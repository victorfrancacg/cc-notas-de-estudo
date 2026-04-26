---
title: VAX/VMS
tags:
  - concept
  - ostep
  - memory
  - case-study
  - history
aliases:
  - VAX/VMS Virtual Memory
  - VMS
type: concept
introduced_in: Ch 23
---

# VAX/VMS

> [!abstract] Definition
> **VAX/VMS** (1977, Digital Equipment Corporation, primary architect Dave Cutler — later of Windows NT) is a foundational virtual-memory operating system. Its [[Memory Virtualization|VM design]] established many ideas still in use today: [[Demand Zeroing|demand zeroing]], [[Copy-On-Write|copy-on-write]], segmented FIFO replacement with second-chance lists, [[Page Cache|clustering of writes]], and reference-bit emulation via protection bits.

## Hardware

- **32-bit VA**, **512-byte pages** — small even for the era; sized for backwards compatibility, not VM efficiency.
- VA layout: `[2-bit segment | 21-bit VPN | 9-bit offset]`.
- Three segments:
  - **P0** — process code + heap (lower half, growing up).
  - **P1** — stack (lower half, growing down).
  - **S** — system/kernel (upper half, shared across processes).
- Hybrid paging+segmentation: per-segment page tables. Saves space when segments are small.

## Address-Space Layout

```
0          ┌────────────┐ Page 0: INVALID (null-pointer trap)
           │ User Code  │
           │ User Heap  │ ← P0
2^30       ├────────────┤
           │  (gap)     │
           │ User Stack │ ← P1 (grows down)
2^31       ├────────────┤
           │ Trap Tables│
           │ Kernel Data│
           │ Kernel Code│ ← S (shared)
           │ Kernel Heap│
2^32       └────────────┘
```

> [!tip] Page 0 = invalid is a 1970s gift to debugging
> Null-pointer dereferences fault cleanly instead of silently corrupting. Modern Linux preserves this convention.

## Innovation 1: Reference Bit Emulation

VAX hardware **lacked a use bit** in PTEs. VMS emulated it (Babaoglu & Joy, 1981) using protection bits:

1. Mark page inaccessible.
2. On access → trap → OS notes "referenced", restores permissions, returns.
3. Periodically re-mark to retest.

The trap overhead was non-trivial but the resulting "approximate LRU" worked. A canonical example of OS software covering for hardware shortcomings.

## Innovation 2: Segmented FIFO + Second-Chance Lists

Per-process **resident set size (RSS)**: a quota of pages each process keeps in RAM. When the process exceeds RSS, its **oldest** page (FIFO) is evicted — *its own*, not anyone else's (**local replacement**).

Evicted pages don't immediately leave physical memory. They go to **global second-chance lists**:
- **Clean** → "clean-page free list" — reclaimable on demand, no I/O.
- **Dirty** → "dirty-page list" — written to disk in batches.

If the original process re-faults on a recently-evicted page, it reclaims the page from these lists at no I/O cost — a "soft" minor fault. The combination of FIFO + second-chance approximates LRU at very low overhead.

## Innovation 3: Clustering of Writes

When dirty pages need to be written to swap, VMS **clusters** many of them and issues large sequential writes. Disk hardware loves this (one seek, many bytes); single-page writes were brutal.

This pattern lives on in every modern OS — Linux's flusher threads do the same.

## Innovation 4: Demand Zeroing

The naive approach: whenever a process asks for a new heap page, find a frame, **zero it** (security: don't leak previous user's data), map it. Slow if the page is never touched.

VMS approach: mark the new page **inaccessible**, flagged "demand-zero". On first access, trap → OS finds frame, zeros, maps, returns. Pay only when used. See [[Demand Zeroing]].

## Innovation 5: Copy-On-Write

When mapping a page into another address space (e.g., on `fork()`), instead of physically copying:

1. Map the page **read-only** into both address spaces.
2. On the first **write** by either side, trap → OS allocates a new frame, copies, marks writable for that side.

Crucial for `fork()` immediately followed by `exec()` (the common case): the entire address space's data needs no copying because `exec()` overwrites it anyway. Lives on in every modern OS. See [[Copy-On-Write]].

## Why VAX/VMS Still Matters

- Many "modern" tricks were already there in 1977.
- Dave Cutler took these ideas to **Windows NT**, where they shaped a generation of OS designs.
- The OSTEP authors highlight VMS as proof that good engineering ages well: 50-year-old design choices remain best-in-class.

## Limitations

- 512 B pages → enormous page tables, lots of TLB misses on big workloads.
- 32-bit VA limited to 4 GB total — exhausted by the early 1990s.
- Hybrid paging+segmentation didn't generalize — modern systems use pure paging.

## Related Notes

- [[Copy-On-Write]] — VMS / TENEX innovation, now universal.
- [[Demand Zeroing]] — VMS innovation, now universal.
- [[Page Replacement]] — VMS's segmented FIFO contrasts with global LRU.
- [[Page Cache]] — Linux's unified equivalent.
- [[Linux Virtual Memory]] — modern counterpart.
- [[Hybrid Paging-Segmentation]] — the architecture VMS built on.
- [[Ch 23 — Complete Virtual Memory Systems]].
