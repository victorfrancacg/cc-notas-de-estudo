---
title: Splitting and Coalescing
tags:
  - concept
  - ostep
  - memory
  - allocation
type: concept
introduced_in: Ch 17
---

# Splitting and Coalescing

> [!abstract] Definition
> The two universal mechanisms that every variable-size memory allocator uses: **splitting** (carve a free chunk down to the requested size, keep the remainder free) and **coalescing** (merge adjacent free chunks back together). Together they keep [[External Fragmentation|external fragmentation]] under some control.

## Splitting

When a free chunk is bigger than needed, the allocator divides it:

```
Before: [free, size 4088]
malloc(100) →
After:  [used, size 100][free, size 3980]   (after subtracting headers)
```

The user gets a pointer to the first sub-chunk; the remainder rejoins the free list.

## Coalescing

When a chunk is freed, the allocator inspects its neighbors:

```
Before: [free 100][used 100][free 100]
free(middle) →
After (no coalesce):  [free 100][free 100][free 100]
After (coalesce):     [free 300]
```

Without coalescing, a 200-byte request would fail despite 300 bytes being free.

> [!tip] Address-ordered free lists make coalescing trivial
> If the list is sorted by start address, adjacency is a `prev_end == this_start` check. Most production allocators do this.

## Why You Can't Skip Coalescing

```
After many alloc/free cycles, no coalescing:
[free 80][used 16][free 80][used 16][free 80][used 16][free 80]...
total free = huge, but no contiguous chunk for a 100-byte request.
```

This is the textbook scenario for "fragmentation kills the program despite plenty of free memory". The fix is always coalesce-on-free.

## When to Coalesce

- **Eager (on every `free`)** — simple, prevents fragmentation buildup, costs a bit per call.
- **Lazy (deferred)** — collect freed blocks, coalesce in a batch later. Cheaper per-call, but heap can be temporarily fragmented.
- **None** — only viable for short-lived programs that exit before exhaustion.

Most general-purpose allocators are eager. Specialty workloads (e.g., region-based allocation) avoid `free` altogether — release the whole arena at once.

## Where Splitting and Coalescing Don't Apply

- **Fixed-size allocators** ([[Paging]], slab object caches) — no splitting, no coalescing, just push/pop on a free list.
- **Buddy allocator** — splitting and coalescing both reduce to power-of-two arithmetic; the buddy bit identifies the merge partner.

## Related Notes

- [[Free-Space Management]] — the umbrella.
- [[Allocation Strategy]] — chooses the chunk to split.
- [[External Fragmentation]] — what coalescing fights.
- [[Buddy Allocator]] — power-of-two splitting/coalescing.
- [[malloc()]], [[free()]] — the API where these mechanisms live.
- [[Ch 17 — Free-Space Management]].
