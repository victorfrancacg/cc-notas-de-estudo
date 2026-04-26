---
title: Demand Paging
tags:
  - concept
  - ostep
  - memory
  - paging
type: concept
introduced_in: Ch 21
---

# Demand Paging

> [!abstract] Definition
> **Demand paging** is the policy of bringing a [[Page|page]] into physical memory **only when it is first accessed**, not eagerly at allocation time. Combined with [[Page Fault|page faults]] for the lazy fetch, it is the dominant strategy in every modern OS — it minimizes useless I/O, reduces startup time, and only consumes physical memory for pages the program actually uses.

## The Idea

A program calls `mmap(NULL, 1 GB, ...)` to allocate 1 GB of virtual memory. The OS does **almost nothing**:

- It records the mapping in the process's VMA list.
- It does **not** allocate any physical frames.
- It does **not** zero or read anything.

Only on first access to each page does a [[Page Fault|page fault]] fire, and only then does the kernel do the work: find a free frame, zero it (anonymous) or read from file/swap, install the PTE.

## What This Buys You

- **Fast `mmap`/`malloc`** — gigabyte allocations return instantly because no real work is done.
- **Memory only for what you touch** — programs commonly allocate buffers larger than they fill; the unused tail costs nothing.
- **Fast process startup** — `exec()` doesn't load the whole binary; the kernel maps the executable file and lets demand paging fault pages in as the program runs.
- **Shared-library efficiency** — many processes mapping `libc.so` only fault in the few thousand instructions they actually use, sharing physical pages.

## Variants and Tweaks

### Demand Zeroing

Anonymous pages start logically as zero. The kernel doesn't zero them up front — it allocates a zeroed frame on first fault. Some kernels keep a pool of pre-zeroed pages (zeroed by an idle thread) so the fault path doesn't have to zero on the critical path.

### Pre-Faulting / Pre-Paging

Sometimes the OS *does* eagerly load pages — typically because access patterns are predictable:

- **Read-ahead** for sequential file mmaps.
- `mmap(MAP_POPULATE)` — explicitly populate now.
- `madvise(MADV_WILLNEED)` — programmer hint.

Pre-faulting trades startup latency for fewer mid-execution faults.

### Lazy vs Eager Backing

- **Lazy** (demand): allocate frame on first access. Default.
- **Eager**: allocate everything up front. Used for real-time / latency-sensitive code where mid-execution faults are intolerable (`mlock()`, `mlockall()`).

## Why It Works (and When It Doesn't)

Demand paging exploits the **principle of locality** — programs touch a small subset of memory at any moment. The page-fault handler is invoked, but most of the time only on cold pages, which would cost I/O anyway.

> [!warning] When demand paging hurts: bursty, unpredictable access
> A short-running, latency-critical program that reads many uncached pages can pay heavily for first-touch faults. Mitigations: pre-faulting, locked memory, or a warm-up pass.

## Related Notes

- [[Page Fault]] — the mechanism demand paging relies on.
- [[Page-Fault Handler]] — runs on each first touch.
- [[Swap Space]] — where major-fault content comes from.
- [[mmap()]] — the API typically used for demand-paged regions.
- [[Page Replacement]] — the eviction half of the picture.
- [[Spatial Locality]], [[Temporal Locality]] — why first-touch costs are tolerable.
- [[Ch 21 — Beyond Physical Memory - Mechanisms]].
