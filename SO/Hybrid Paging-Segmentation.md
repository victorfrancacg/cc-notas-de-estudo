---
title: Hybrid Paging-Segmentation
tags:
  - concept
  - ostep
  - memory
  - paging
  - segmentation
aliases:
  - Paged Segmentation
  - Hybrid Memory Scheme
type: concept
introduced_in: Ch 20
---

# Hybrid Paging-Segmentation

> [!abstract] Definition
> A historical scheme — pioneered by Multics, used in VAX/VMS — that combines [[Paging]] and [[Segmentation]]: each segment of the address space (code, heap, stack) gets its **own** [[Page Table]]. Segment registers hold the physical address of the segment's page table plus a bound on its size. Reduces page-table waste vs a flat per-process table, but inherits segmentation's external-fragmentation problem and assumes the address space breaks into a few segments.

## The Translation Path

```
VA  = [ SN  | VPN | offset ]

SN     = top bits → which segment
VPN    = next bits → page within segment
offset = low bits → byte within page

Base[SN]   = physical address of this segment's page table
Bound[SN]  = number of valid PTEs in this segment's page table

PTEAddr = Base[SN] + (VPN * sizeof(PTE))
if VPN >= Bound[SN]:  raise(SEGFAULT)
```

So each segment register is now a (base, bound) pair pointing at a **page table**, not at the segment data itself.

## Why It Saved Memory

A 32-bit linear page table costs 4 MB even if the heap is 1 MB. With a hybrid scheme:

- **Code page table** sized to actual code (say 256 entries → 1 KB).
- **Heap page table** sized to actual heap (say 256 entries).
- **Stack page table** small.

Empty space *between* segments costs nothing — no PTE entries need exist for it.

## The Problems

> [!warning] Still bound to segmentation's assumptions
> A "large but sparsely used heap" still wastes its segment's page table. Hybrid only helps when the address space neatly factors into a few segments.

> [!warning] Variable-sized page tables → external fragmentation again
> Page tables are now of arbitrary size (in multiples of PTEs). The OS must find space for them in physical memory — and segments-grow / segments-shrink reproduce the [[External Fragmentation|external-fragmentation]] problem in **page-table memory**.

## Why Modern Systems Skipped This

[[Multi-Level Page Table|Multi-level page tables]] solve the same problem (memory ∝ usage, not max size) **without** assuming the address space factors into segments. Multi-level tables are the universal modern answer; hybrid schemes are mostly historical curiosities (VAX/VMS being the canonical example).

x86 *kept* segment registers for legacy reasons but in 64-bit mode they're flattened — the modern OS uses segments only for thread-local-storage trickery (`FS`, `GS`).

## Related Notes

- [[Segmentation]] — one half of the hybrid.
- [[Paging]] — the other half.
- [[Multi-Level Page Table]] — the modern replacement.
- [[Page Table]] — what hybrid replicates per segment.
- [[External Fragmentation]] — what the variable-size tables resurrect.
- [[Ch 20 — Paging - Smaller Tables]].
