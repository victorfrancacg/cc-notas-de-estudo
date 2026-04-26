---
title: Memory Compaction
tags:
  - concept
  - ostep
  - memory
  - fragmentation
aliases:
  - Compaction
type: concept
introduced_in: Ch 16
---

# Memory Compaction

> [!abstract] Definition
> **Compaction** is the technique of physically rearranging allocated memory regions to merge scattered free holes into one contiguous free area. It's a heavy hammer for fixing [[External Fragmentation|external fragmentation]] — effective, but costly: every byte that moves is read and written.

## How It Works

```
Before                  After
0   ┌─────────┐ OS    0   ┌─────────┐ OS
16  ├─────────┤       16  ├─────────┤
    │ (free)  │           │ Alloc 1 │
24  ├─────────┤       24  │         │
    │ Alloc 1 │       32  ├─────────┤
32  ├─────────┤           │ Alloc 2 │
    │ (free)  │       40  ├─────────┤
40  ├─────────┤           │ Alloc 3 │
    │ Alloc 2 │       48  ├─────────┤
48  ├─────────┤           │ (free)  │
    │ (free)  │           │         │
56  ├─────────┤           │         │
    │ Alloc 3 │       64  └─────────┘
64  └─────────┘
```

The OS:

1. Pauses (or quiesces) running processes.
2. Copies allocated regions toward one end of physical memory.
3. Updates each process's segment registers / page tables to point to the new physical addresses.
4. Resumes execution.

After compaction the free space is contiguous; the next big allocation succeeds.

## Costs

- **Memory bandwidth** — copying every allocated byte. For a system with tens of GB of allocations, this dominates.
- **Latency / pauses** — the world stops for the affected processes.
- **Pointer fix-ups** — if the language exposes physical pointers, they may not be relocatable. Garbage-collected languages with handle indirection (or relocating GCs) compact freely; C-like systems can't compact heap allocations because pointers held by the program would silently break.

## Where Compaction Is Used

- **GC heaps** (Java's mark-compact GC, Go's compacting GC) — the runtime owns all references and can update them after moving.
- **Defragmenting filesystems** (offline `e4defrag`, NTFS defrag) — filesystem analog.
- **Multics-era OSes** with full segmentation — historical.

## Where Compaction Is Avoided

- **C/C++ heaps** (`malloc`/`free`) — the program holds raw pointers to allocated chunks. Moving them silently invalidates those pointers. So `malloc` allocators rely on **smart free-list policies** + **coalescing of adjacent frees** instead.
- **Paged virtual memory** — there's nothing to compact; pages are fixed size and external fragmentation doesn't exist at the page granularity.

## Related Notes

- [[External Fragmentation]] — what compaction fixes.
- [[Internal Fragmentation]] — what it doesn't fix.
- [[Segmentation]] — the primary OSTEP context for compaction.
- [[Free-Space Management]] — Ch 17, the cheaper alternative for heaps.
- [[Paging]] — sidesteps the need for compaction entirely.
- [[Ch 16 — Segmentation]].
