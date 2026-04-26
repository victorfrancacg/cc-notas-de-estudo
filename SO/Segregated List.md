---
title: Segregated List
tags:
  - concept
  - ostep
  - memory
  - allocation
aliases:
  - Segregated Lists
  - Size-Class List
type: concept
introduced_in: Ch 17
---

# Segregated List

> [!abstract] Definition
> A **segregated list** allocator maintains **separate free lists per popular allocation size** (or size class). Requests of that size hit a fast O(1) path; everything else falls through to a general allocator. The pattern underlies the kernel's [[Slab Allocator|slab allocator]] and the user-space allocators jemalloc and tcmalloc.

## The Idea

If your workload allocates lots of 32-byte and 256-byte objects, why search a general free list every time? Keep one list **per popular size**:

```
list[16B]:   [free][free][free]
list[32B]:   [free][free][free][free]
list[64B]:   [free]
list[128B]:  []
list[256B]:  [free][free][free][free][free][free]
overflow:    general allocator (best-fit / buddy)
```

`malloc(32)` → pop from `list[32B]`. O(1).

## Where Sizes Come From

- **Slab caches** — one cache per kernel object type (inode, lock, socket).
- **jemalloc / tcmalloc** — fixed size classes (8, 16, 32, 48, 64, 80, 96, …) covering small allocations; large requests bypass.
- **Per-thread arenas** — each thread keeps its own segregated lists to dodge lock contention.

## Why It Wins

| Pro | Con |
|---|---|
| O(1) alloc/free for popular sizes | Memory dedicated to a size class can sit unused |
| No fragmentation within a class | Need to choose size classes wisely |
| Excellent locality (same-size objects clustered) | Complex bookkeeping |

## Modern User-Space Reality

The default `malloc` in glibc, jemalloc, and tcmalloc all use segregated lists for small allocations and fall back to other strategies (best-fit trees, `mmap`) for large allocations. **The fast path is segregated; the slow path is structural.**

## Related Notes

- [[Slab Allocator]] — the kernel-side segregated-list allocator.
- [[Buddy Allocator]] — alternative structural scheme; often used together.
- [[Free-Space Management]] — the umbrella concept.
- [[Allocation Strategy]] — segregated lists sidestep best/worst/first-fit.
- [[Ch 17 — Free-Space Management]].
