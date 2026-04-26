---
title: Sparse Address Space
tags:
  - concept
  - ostep
  - memory
type: concept
introduced_in: Ch 16
---

# Sparse Address Space

> [!abstract] Definition
> An [[Address Space]] is **sparse** when most of it is unused: large gaps separate the few regions actually populated (code, heap, stack, mapped files, libraries). A 64-bit address space (16 exabytes) is *always* sparse — only a tiny fraction can ever be physically backed.

## Why Address Spaces Are Sparse

- **Stack vs heap layout** — placing them at opposite ends leaves a huge middle gap.
- **Multiple thread stacks** — each in its own region, with unused space between.
- **Shared libraries** — mapped at separate (often randomized via ASLR) base addresses; gaps everywhere.
- **`mmap`'d files / anonymous regions** — scattered through the address space.
- **Address-space size ≫ physical RAM** — even fully-used 64-bit address spaces couldn't be backed.

## Why It Matters

> [!warning] [[Base and Bounds]] can't handle it
> A single base/bounds pair must cover the *entire* used range, including the giant gap. Wasted physical memory equals the gap size.

> [!tip] [[Segmentation]] handles it
> Per-segment base/bounds. Only used segments occupy physical memory; gaps cost zero physical bytes.

> [!tip] [[Paging]] handles it via on-demand allocation
> Pages are allocated only when first touched. Unused pages of the address space have no physical backing, no [[Page Table]] entries that need physical frames (with [[Multi-Level Page Table|multi-level tables]]).

## Modern Address Spaces Are Extremely Sparse

A 64-bit Linux process might have a virtual address space of 2⁴⁸ ≈ 256 TB but actually use ~50 MB. That's *trillionths* of a percent — sparseness is the norm, not the exception.

This sparseness is one of the main reasons every modern OS uses paging plus multi-level (or inverted) page tables, and why VM-system design has spent decades getting good at sparse representation.

## Related Notes

- [[Address Space]] — the container.
- [[Segmentation]] — handles sparseness via per-segment base/bounds.
- [[Paging]] — handles sparseness via demand-paged frames.
- [[Multi-Level Page Table]] — sparseness-aware page-table representation.
- [[mmap()]] — fills the address space with discontiguous mappings.
- [[Ch 16 — Segmentation]].
