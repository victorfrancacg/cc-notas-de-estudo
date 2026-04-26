---
title: Segmentation
tags:
  - concept
  - ostep
  - memory
  - virtualization
  - mechanism
type: concept
introduced_in: Ch 16
---

# Segmentation

> [!abstract] Definition
> **Segmentation** is the address-translation scheme in which the [[Address Space]] is divided into a few logical **segments** (typically code, heap, stack), each with its own [[Base and Bounds|base/bounds]] pair in the [[MMU]]. Generalizes single-pair base/bounds; eliminates [[Internal Fragmentation|internal fragmentation]] *within the address space*; introduces [[External Fragmentation|external fragmentation]] in physical memory.

## Per-Segment Translation

For each access:

```
seg     = which segment? (top bits, or implicit from access type)
offset  = address minus segment-start (or top bits stripped)
if (offset >= bounds[seg]) trap
physical = base[seg] + offset
check protection_bits[seg] against access type (R/W/X)
```

## What Each Segment Holds

- **Code** segment — read-execute; shareable across processes.
- **Heap** segment — read-write; grows up.
- **Stack** segment — read-write; grows **down** (negative growth).

## Strengths

- **Sparse address spaces** — the vast unused gap between heap and stack costs zero physical memory.
- **Sharing** — a read-only code segment can map into multiple address spaces, sharing one physical copy.
- **Per-segment protection** — read-only code, non-executable stack (modern equivalent: `NX` bit / W^X).

## Weaknesses

> [!warning] [[External Fragmentation]]
> Variable-sized segments fragment physical memory. A 20 KB request can fail despite 24 KB free, if it's split into smaller holes.

> [!warning] Growing segments are awkward
> A heap that runs out of space requires either in-place growth (only possible if there's adjacent free physical memory) or relocation (copy the segment elsewhere, update base).

> [!warning] Coarse-grained granularity
> Three segments per process is fine; thousands (Multics-style) requires a **segment table** in memory and burns bookkeeping. Modern systems mostly skipped this and went straight to [[Paging]].

## Where It Lives Today

x86 had segment registers (`CS`, `DS`, `ES`, `SS`, `FS`, `GS`) that everyone used until paging took over. In 64-bit mode most of them are flattened (base = 0, no bounds) and the architecture *de facto* runs paging-only — except `FS` and `GS`, still used for thread-local storage by glibc.

ARM, RISC-V, MIPS skipped segments entirely. So in modern OS practice **segmentation is historically interesting but rarely used as a primary mechanism**.

## Related Notes

- [[Ch 16 — Segmentation]] — full chapter treatment.
- [[Base and Bounds]] — what segmentation generalizes.
- [[External Fragmentation]] — its main downside.
- [[Sparse Address Space]] — what it enables.
- [[Paging]] — the eventual replacement.
- [[Memory Compaction]] — one mitigation strategy.
- [[Segmentation Fault]] — the eponymous signal.
