---
title: External Fragmentation
tags:
  - concept
  - ostep
  - memory
  - fragmentation
type: concept
introduced_in: Ch 16
---

# External Fragmentation

> [!abstract] Definition
> **External fragmentation** is wasted space *between* allocated units in physical memory (or in the heap). The total free space may be large, but it's split into many small holes — no single contiguous region is big enough to satisfy a new variable-sized request. Distinct from [[Internal Fragmentation|internal fragmentation]] (waste *inside* a unit).

## Where It Appears in OSTEP

### Segmentation (Ch 16)

Variable-sized segments allocated and freed in physical memory leave gaps:

```
0KB  ┌────────────┐ OS
8KB
16KB ├────────────┤
     │ (free)     │
24KB ├────────────┤
     │ Allocated  │
32KB ├────────────┤
     │ (free)     │
40KB ├────────────┤
     │ Allocated  │
48KB ├────────────┤
     │ (free)     │
56KB ├────────────┤
     │ Allocated  │
64KB └────────────┘
```

24 KB total free, split across 3 holes. A 20 KB request fails because no contiguous 20 KB chunk exists.

### Heap allocators (Ch 17)

`malloc`/`free` over time produces a free list with many small entries. Variable-sized requests increasingly fail despite plenty of total free bytes.

## Mitigations

| Strategy | What it does | Cost |
|---|---|---|
| **[[Memory Compaction\|Compaction]]** | Move allocated chunks to coalesce free space | Memory-bandwidth-heavy; pauses the system |
| **Smart free-list policies** | best-fit, worst-fit, first-fit, next-fit, buddy | Heuristic; never eliminates frag |
| **Coalescing** | Merge adjacent free blocks at `free` time | Cheap; helps a lot |
| **Fixed-size units** ([[Paging]]) | Allocate only equal-size pieces | Solves it; introduces internal frag |

> [!warning] No allocator can eliminate external fragmentation under variable-sized requests
> Decades of research (Wilson et al., 1995 survey) confirm there's no policy that wins across all workloads. The clean fix is to abandon variable sizes — i.e., switch to [[Paging]].

## External vs Internal — Quick Recap

```
INTERNAL                 EXTERNAL
┌────────────┐          ┌──┐ ┌────┐ ┌──┐
│ used│waste │           │A │ │ B  │ │C │
└────────────┘          └──┘ └────┘ └──┘
                          ↑      ↑
                      free   free (small holes,
                              total > request,
                              none big enough)
```

Paging eliminates external (everything is a fixed page) and introduces internal (last page of each allocation may be partially used). Segmentation eliminates internal (within an address space) and introduces external (physical memory).

## Related Notes

- [[Internal Fragmentation]] — the dual concept.
- [[Segmentation]] — primary source of external fragmentation in OSTEP.
- [[Free-Space Management]] — Ch 17, the policies that mitigate it.
- [[Memory Compaction]] — one mitigation.
- [[Paging]] — the structural escape from external fragmentation.
- [[Ch 16 — Segmentation]].
