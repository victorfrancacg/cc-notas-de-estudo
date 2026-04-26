---
title: Linux Virtual Memory
tags:
  - concept
  - ostep
  - memory
  - case-study
  - linux
type: concept
introduced_in: Ch 23
---

# Linux Virtual Memory

> [!abstract] Definition
> The **Linux virtual memory system** is the modern, feature-rich VM stack used in everything from phones to hyperscale datacenters. It combines hardware-walked multi-level [[Page Table|page tables]] (x86_64 four-level), a unified [[Page Cache|page cache]] for files and swap, [[Huge Pages|huge-page support]], and a security stack ([[ASLR]], NX bit, KASLR, KPTI) shaped by decades of attacks.

## Address Space

```
0x00000000  ┌────────────┐ Page 0 invalid
            │ User Code  │
            │ User Heap  │
            │   ...      │ User space
            │ User Stack │
0xC0000000  ├────────────┤    (32-bit; 64-bit splits much higher)
            │ Kernel     │
            │ (Logical)  │ Kernel space, shared across all processes
            │ Kernel     │
            │ (Virtual)  │
0xFFFFFFFF  └────────────┘
```

The kernel half is mapped into **every** process's address space. Syscalls don't need to switch tables — just elevate privilege. (Until KPTI; see below.)

## Kernel Logical vs Kernel Virtual Memory

Linux distinguishes two kinds of kernel addresses:

| | Kernel logical | Kernel virtual |
|---|---|---|
| Virtual-to-physical mapping | Direct (offset only) | Through page tables |
| Allocator | `kmalloc()` | `vmalloc()` |
| Physically contiguous? | Always | Not necessarily |
| Suitable for DMA? | Yes | No |
| Swappable? | No | Yes (rarely used) |
| Use case | Page tables, per-CPU stacks, kernel data structures | Large buffers where contig is hard |

The first 1 GB (on 32-bit) is direct-mapped 1:1 with physical RAM, supporting DMA and avoiding per-access page-table walks for kernel internals.

## Page Tables

x86 hardware-managed multi-level:
- 32-bit: 2-level (or 3-level with PAE).
- 64-bit: **4-level** (top 16 bits of VA unused; 9+9+9+9+12 = 48 bits).
- 5-level on newer chips for larger VAs.

OS sets up tables, points `CR3` (the page-table base register) at the current process's table, hardware walks on every TLB miss.

## Huge Pages

Three page sizes simultaneously:

- **4 KB** — default.
- **2 MB** — huge pages.
- **1 GB** — gigantic pages.

Two API modes:

- **Explicit** — `mmap(MAP_HUGETLB)` / `shmget(SHM_HUGETLB)`. Apps opt in.
- **Transparent (THP)** — kernel automatically promotes contiguous 4 KB regions to 2 MB when possible. Best for general workloads.

See [[Huge Pages]] for the trade-offs.

## The Page Cache

> [!info] Unified for everything disk-backed
> File pages, anonymous pages destined for swap, block-device metadata — **one** cache, **one** replacement policy, **one** eviction loop.

Pages are tracked as **clean** or **dirty**. Dirty pages get written back periodically by flusher threads (`pdflush` historically; now `flusher` per backing device).

Replacement: a **modified 2Q** with active and inactive lists:

- New page → inactive list.
- Re-referenced → promoted to active list.
- Eviction always from the inactive tail.
- Periodically rebalances active ↔ inactive.

Scan-resistant by construction: a one-time large file read flows through the inactive list without disturbing the hot active list.

## Memory Mapping Is Everywhere

Every Linux process is a **patchwork of memory-mapped regions**:

```
$ pmap <pid>
0000000000400000   372K r-x--  /usr/bin/tcsh
00000000019d5000  1780K rw---  [anon]            ← heap
00007f4e7cf06000  1792K r-x--  /lib/libc.so
00007f4e7d2d0000    36K r-x--  /lib/libcrypt.so
...
00007f4e7d932000    16K rw---  [stack]
```

Code, libraries, dynamic linker — all `mmap`-backed from the file system. Heap and stack — anonymous mappings. There's no "load the whole binary" step; pages fault in lazily.

## Security Stack

### NX bit (No-Execute)

Per-page flag: pages marked non-executable trap on instruction fetch. Defeats classic shellcode injection on the stack/heap.

### ASLR (Address Space Layout Randomization)

Code, libraries, stack, heap base addresses randomized at process start. Attacks can't hard-code targets. **KASLR** does the same for kernel mappings.

### Return-Oriented Programming (ROP)

The attackers' response to NX: chain existing code fragments ("gadgets") via a corrupted stack. ASLR makes ROP harder by randomizing where the gadgets are.

### Meltdown / Spectre (2018)

Speculative execution leaves cache traces even when rolled back. Attackers can read other processes' or the kernel's memory by training the CPU and timing cache.

### KPTI (Kernel Page-Table Isolation)

The Meltdown countermeasure: kernel gets its own page table. Switching to/from kernel now swaps tables (slower) but prevents speculative kernel-memory leak.

> [!warning] KPTI undid a foundational optimization
> Linux had mapped all of the kernel into every process's upper half *for performance* (no syscall page-table swap). Meltdown forced reversal: now syscalls cost more, but the kernel is hidden from user-space speculation.

## Other Niceties

- **Demand zeroing** — anonymous pages allocated lazily; first touch zeros and maps. See [[Demand Zeroing]].
- **Copy-on-write** — `fork()` shares pages until either side writes. See [[Copy-On-Write]].
- **`/dev/zero`** — file-backed source of zero-pages, used by the demand-zero machinery.
- **`swapd` background thread** — swaps cold pages out under memory pressure; not the foreground fault handler's job.

## Related Notes

- [[VAX-VMS]] — the classical predecessor.
- [[Page Cache]] — unified file-and-swap cache.
- [[Huge Pages]] — TLB-coverage win.
- [[Copy-On-Write]] — `fork()` would be hopeless without it.
- [[Demand Zeroing]] — every anonymous page benefits.
- [[ASLR]] — security baseline.
- [[Multi-Level Page Table]] — x86_64's structure.
- [[mmap()]] — the universal mapping primitive.
- [[Ch 23 — Complete Virtual Memory Systems]].
