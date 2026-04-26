---
title: Internal Fragmentation
tags:
  - concept
  - ostep
  - memory
  - fragmentation
type: concept
introduced_in: Ch 15
---

# Internal Fragmentation

> [!abstract] Definition
> **Internal fragmentation** is wasted space *inside* an allocated unit. The allocator gave you more bytes than you asked for (for alignment, fixed-size slots, or rounding), and the unused tail is unreachable until the entire unit is freed. Distinct from [[External Fragmentation|external fragmentation]] (gaps *between* allocated units).

## Where It Appears in OSTEP

### Base and Bounds (Ch 15)

A process gets a single fixed-size slot. If the slot is 16 KB but the process only uses 4 KB of code/heap/stack, the remaining 12 KB sits *inside* the slot — wasted. This is the textbook example.

### Paging (Ch 18)

The unit of allocation is a **page** (e.g., 4 KB). A 4097-byte allocation costs **two** pages = 8 KB. The trailing 4095 bytes of the second page are internal fragmentation.

> [!tip] Bigger pages → more internal fragmentation
> 1 MB pages reduce TLB pressure but waste up to 1 MB per allocation that doesn't fill a page. Always a trade-off; modern systems mix sizes (huge pages + 4 KB pages).

### Heap allocators (Ch 17)

`malloc(13)` may actually consume 16 bytes (rounded up for alignment) plus a small header. The 3 unused bytes inside your block are internal fragmentation. Headers themselves count as overhead — sometimes called bookkeeping fragmentation.

## Internal vs External

```
INTERNAL                EXTERNAL
┌────────────┐         ┌──┐  ┌────┐  ┌──┐
│ used│ wasted│         │A │  │ B  │  │C │
└────────────┘         └──┘  └────┘  └──┘
                          ↑       ↑
                       free    free  ← total free is large but
                                       no contiguous chunk fits
                                       a 6-byte request
```

You can have one without the other. [[Paging]] eliminates external but introduces internal; [[Segmentation]] eliminates internal (within a process) but suffers external.

## Mitigation

- **Smaller units** — finer-grained slots reduce per-unit waste, at the cost of more bookkeeping.
- **Multiple page sizes** — huge pages where they help, small pages where they don't.
- **Slab allocators** — pools of equal-sized objects (kernel uses this).
- **Compaction** — only useful for external; doesn't help internal.

## Related Notes

- [[External Fragmentation]] — the dual problem.
- [[Base and Bounds]] — generates internal fragmentation per process.
- [[Paging]] — generates internal fragmentation per page.
- [[Free-Space Management]] — Ch 17, where allocator-level internal frag lives.
- [[Memory Compaction]] — a fix for external, not internal.
- [[Ch 15 — Mechanism - Address Translation]].
