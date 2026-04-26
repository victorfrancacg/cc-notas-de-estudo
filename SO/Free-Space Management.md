---
title: Free-Space Management
tags:
  - concept
  - ostep
  - memory
  - allocation
type: concept
introduced_in: Ch 17
---

# Free-Space Management

> [!abstract] Definition
> **Free-space management** is the discipline of tracking unused memory and choosing where to satisfy each variable-sized allocation request. It lives at two layers: inside [[malloc()|malloc]]/[[free()|free]] (managing a process's heap) and inside the OS (managing physical memory under [[Segmentation]] or other variable-size schemes).

## The Two Layers

| Where | Managing | Granularity |
|---|---|---|
| `malloc`/`free` | a process's [[Heap (Runtime)\|heap]] | bytes (with header overhead) |
| OS allocator | physical memory under segmentation | KB segments |
| OS allocator | physical frames under paging | fixed page size — **[[Free-Space Management]] becomes trivial** |

## Mechanisms (Universal)

- **[[Splitting and Coalescing]]** — carve free chunks down on alloc; merge adjacent free chunks on free.
- **Embedded free list** — free-chunk metadata lives inside the free chunks themselves.
- **Header words** — every allocated block has a small header recording its size (and often a sanity-check magic number) so `free` can find the size.

## Policies

See [[Allocation Strategy]] for full coverage. The four classics:

- **Best-fit** — smallest chunk ≥ request.
- **Worst-fit** — largest chunk; leave remainder.
- **First-fit** — first chunk that fits.
- **Next-fit** — like first-fit but resume from last position.

Plus structural alternatives: [[Buddy Allocator|buddy]], [[Slab Allocator|slab]], [[Segregated List|segregated lists]].

## Why It Matters

> [!warning] Variable-sized allocation never has a clean answer
> Decades of allocator research show no policy that wins on all workloads. The pragmatic move is to **avoid variable-sized allocation entirely**: hence [[Paging|paging]], slab pools for kernel objects, and arena allocators for short-lived workloads.

## Pitfalls

- **Forgetting to coalesce** — leaves the heap fragmented even when fully freed.
- **Linear free-list scans** — a million-entry free list with best-fit becomes the bottleneck. Modern allocators use trees or per-size lists.
- **Lock contention** in multithreaded programs — solved with per-thread arenas (jemalloc, tcmalloc).

## Related Notes

- [[Allocation Strategy]] — best/worst/first/next-fit details.
- [[Splitting and Coalescing]] — mechanisms.
- [[Buddy Allocator]], [[Slab Allocator]], [[Segregated List]] — alternative structures.
- [[External Fragmentation]] — what variable-sized allocation creates.
- [[Internal Fragmentation]] — what fixed-size allocation creates instead.
- [[Paging]] — the structural escape.
- [[Ch 17 — Free-Space Management]].
