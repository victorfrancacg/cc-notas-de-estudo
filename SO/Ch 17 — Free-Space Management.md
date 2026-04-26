---
title: Ch 17 — Free-Space Management
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
type: chapter
book: OSTEP
chapter: 17
pages: 181-198
---

# Ch 17 — Free-Space Management

> [!abstract] One-sentence summary
> When a memory manager (user-level [[malloc()|malloc]] or kernel-level segment allocator) hands out **variable-sized** chunks, [[External Fragmentation|external fragmentation]] is the central enemy. The chapter walks the **mechanisms** (splitting, coalescing, free list embedded in the free space, header words with a magic number) and the **policies** (best-fit, worst-fit, first-fit, next-fit, segregated/[[Slab Allocator|slab]] lists, [[Buddy Allocator|buddy]]). No policy wins universally.

## Crux

> [!example] The Crux: How To Manage Free Space
> How should free space be managed when satisfying variable-sized requests? What strategies minimize fragmentation? What are the time and space overheads of alternatives?

## 17.1 Assumptions

The chapter constrains itself for clarity:

- **Variable-sized requests** (otherwise free-space management is trivial — see fixed-size paging).
- **No compaction** — once handed out, a chunk can't be moved (C programs hold raw pointers).
- **Single contiguous region** — the heap, of fixed initial size; growth happens via the OS ([[brk() and sbrk()|brk/sbrk]] or [[mmap()]]).
- Both **internal** and **external** fragmentation possible; chapter focuses on external.

## 17.2 Low-Level Mechanisms

### Splitting and Coalescing

```
Initial free list: [free 0..9] → [free 20..29]
Request: 1 byte
After split: [free 0..0 used] [free 1..9] → [free 20..29]
```

**Splitting** — when a free chunk is bigger than the request, carve off only what's needed; leave the rest free.

**Coalescing** — when freeing a chunk, check if neighbors are also free; if so, merge into one bigger chunk. Without coalescing, repeated alloc/free leaves the heap fragmented even when *all* memory is technically free:

```
After 3 frees, no coalescing:
[free 100][used][free 100][used][free 100][used][free 3764]
                                   ↑
                        a 200-byte request fails
```

> [!tip] Coalesce on `free`
> Most allocators coalesce eagerly: at every `free()`, walk the free list to find adjacent free blocks and merge. This is why **address-ordered free lists** are popular — adjacency is a `next == this + size` check.

### Tracking Allocated Region Sizes — the Header Word

```c
typedef struct {
    int   size;
    int   magic;        // sanity-check sentinel, e.g., 0x1234567
} header_t;
```

Every allocated block has a header *just before* the user pointer:

```
        ┌──── header ────┐
        │ size │ magic   │
        ├─────────────────┤
ptr ──→ │ user bytes      │   ← what malloc returns
        └─────────────────┘
```

`free(ptr)` walks back one `header_t` to find the size and verify the magic. The user requesting `N` bytes really costs `N + sizeof(header_t)`.

### Embedding the Free List Inside the Free Space

The allocator can't `malloc` to make free-list nodes — it *is* the malloc. So the nodes live **inside the free chunks themselves**:

```c
typedef struct __node_t {
    int               size;
    struct __node_t  *next;
} node_t;
```

When the chunk is allocated, those bytes become user data; when free, they're list metadata. Recycling is the whole game.

### Growing the Heap

When the heap fills up, the allocator either:
- Returns `NULL` (honest failure), **or**
- Asks the OS for more memory via [[brk() and sbrk()|sbrk]] or [[mmap()|anonymous mmap]], extends the free list, and tries again.

## 17.3 Basic Strategies — Allocation Policies

> [!info] None is universally best
> All four are simple, and all four can do badly under adversarial workloads.

### Best Fit
Search the entire free list; return the **smallest** chunk that's ≥ the request.
- ✓ Less wasted space per allocation.
- ✗ Full-list scan every time. Tends to leave many tiny leftovers.

### Worst Fit
Search; return the **largest** chunk; leftover stays large.
- ✓ Tries to keep big chunks available for large future requests.
- ✗ Studies show it actually fragments worse than best-fit. Full-list scan again.

### First Fit
Return the **first** chunk that fits.
- ✓ Fast — no full scan.
- ✗ Pollutes the front of the list with small leftovers. Mitigated by **address-based ordering**.

### Next Fit
Like first-fit, but resume the search from where the last allocation ended (a "rover" pointer).
- ✓ Spreads small leftovers across the list.
- ✓ Fast.
- ≈ Performance comparable to first-fit.

> [!example] Quick comparison
> Free list: 10 → 30 → 20. Allocation request = 15.
> - **Best-fit** picks 20, leaves 5. Result: 10 → 30 → 5.
> - **Worst-fit** picks 30, leaves 15. Result: 10 → 15 → 20.
> - **First-fit** picks 30 (first ≥ 15), leaves 15. Result: 10 → 15 → 20.

## 17.4 Other Approaches

### [[Segregated List|Segregated Lists]]
Maintain separate free lists per popular size. The kernel's [[Slab Allocator|slab allocator]] (Bonwick, 1994) uses this for object caches: file inodes, lock structs, dentries each have their own pool.

### [[Buddy Allocator]]
Conceptually divide free memory into power-of-two-sized regions. To allocate, recursively split the smallest region that fits. To free, check whether the **buddy** (the other half from the last split) is also free; if so, **coalesce** back up.

```
       ┌── 64 KB ──┐
       ├──────────┬┤
       │ 32 KB    │ 32 KB
       ├─────┬────┤
       │16 KB│16 KB│
       ├──┬──┤
       │8K│8K│         ← allocate 8K, leftmost
```

- ✓ Coalescing is cheap: bit manipulation finds the buddy.
- ✗ [[Internal Fragmentation|Internal fragmentation]] — only power-of-two sizes available.
- Used heavily in Linux kernel for physical-page allocation.

### Other Ideas
Balanced trees (red-black, splay), per-CPU caches, lock-free designs — all pursued in real-world allocators (jemalloc, tcmalloc, Hoard) for **scalability** under multithreading.

## 17.5 Summary

> [!example] Crux answered
> No allocator is best for every workload. Mechanisms (splitting, coalescing, headers, embedded free lists) are universal. Policy choice (best/worst/first/next-fit, segregated, buddy) is workload-dependent. Modern allocators add **per-CPU pools** and **size-class pools** to escape both fragmentation and lock contention.

## Aside: Why glibc malloc Looks the Way It Does

[[malloc()]] in glibc is *very* sophisticated: small allocations go into per-thread arenas with size classes; large allocations go directly to `mmap`; coalescing happens at `free`. Reading the "Understanding glibc malloc" article (cited in OSTEP refs) is highly recommended.

## Related Notes

- [[Free-Space Management]] — the standalone concept note.
- [[Allocation Strategy]] — best/worst/first/next-fit summary.
- [[Splitting and Coalescing]] — the universal mechanisms.
- [[Buddy Allocator]], [[Slab Allocator]], [[Segregated List]] — alternative schemes.
- [[malloc()]], [[free()]] — the API consumers.
- [[External Fragmentation]], [[Internal Fragmentation]] — what the chapter fights.
- [[brk() and sbrk()]], [[mmap()]] — how the heap actually grows.
- Next: [[Ch 18 — Paging - Introduction]] — sidesteps these problems with fixed-size units.
- Index: [[MOC - Memory Virtualization]].
