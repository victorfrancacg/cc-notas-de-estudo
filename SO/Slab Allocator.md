---
title: Slab Allocator
tags:
  - concept
  - ostep
  - memory
  - allocation
  - kernel
type: concept
introduced_in: Ch 17
---

# Slab Allocator

> [!abstract] Definition
> The **slab allocator** (Bonwick, 1994; originally for Solaris, now in Linux) is a kernel memory allocator that maintains **per-object-type pools**. Each pool ("cache") holds many pre-initialized objects of a single type (file inodes, dentries, locks, sockets…). Allocation and free are O(1), and freed objects skip re-initialization.

## The Idea

Kernel allocations are **highly repetitive**: the same handful of object types get allocated and freed at huge rates (file descriptors, network buffers, page-table entries). For these:

- Sizes are known and fixed per type.
- Initialization (constructor) cost is non-trivial.
- Deallocation patterns cluster in time.

A general-purpose allocator wastes work on each of these. The slab allocator specializes:

```
Object cache: "kmalloc-256"
┌─────────────────────────────────────┐
│ slab #1: [obj][obj][obj][obj][obj]  │  (some used, some free)
├─────────────────────────────────────┤
│ slab #2: [obj][obj][obj][obj][obj]  │
└─────────────────────────────────────┘
        ↑
   free list of objects
```

A **slab** is a few contiguous physical pages (from the [[Buddy Allocator|buddy allocator]]) carved into objects of one size. The cache tracks which slots are free.

## Allocation Path

1. Look up the cache for the requested object type.
2. Pop a free slot from the cache's free list.
3. Return the (already-initialized) object.

`O(1)`, no fit search, no fragmentation.

## Why "Pre-initialized" Matters

A naive cache zeroes/constructs on every alloc and destructs on every free — wasted work because the same object type immediately gets reconstructed. The slab allocator keeps freed objects **in their already-initialized state**, so reuse is essentially free.

## Trade-Offs

| Pro | Con |
|---|---|
| O(1) alloc/free | Per-object-type pool overhead |
| No fragmentation within a cache | Whole-slab waste if a cache is rarely used |
| Skips repeat init | Memory pinned in caches even if unused |
| Excellent CPU-cache locality | Complex; many tunables |

## Modern Variants

- **SLAB** — original Bonwick design.
- **SLUB** — Linux's current default (simpler, better on multicore).
- **SLOB** — for tiny memory systems (embedded).

## Layered on Top of Buddy

```
+--------------------+   per-object caches
|   slab allocator   |
+--------------------+
|   buddy allocator  |   physical-page allocation
+--------------------+
|     raw RAM        |
```

Slab gets pages from buddy when a cache needs to grow; returns whole slabs to buddy when objects in a slab all become free.

## Related Notes

- [[Buddy Allocator]] — what slab sits on top of.
- [[Free-Space Management]] — kernel-side specialization.
- [[Segregated List]] — slab is a segregated-list scheme par excellence.
- [[Allocation Strategy]] — slab bypasses fit policies entirely.
- [[Ch 17 — Free-Space Management]].
