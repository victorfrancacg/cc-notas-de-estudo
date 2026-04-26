---
title: Paging
tags:
  - concept
  - ostep
  - memory
  - virtualization
  - mechanism
type: concept
introduced_in: Ch 18
detailed_in: Ch 19, 20
---

# Paging

> [!abstract] Definition
> **Paging** is the [[Memory Virtualization|memory-virtualization]] scheme that divides the [[Address Space]] into fixed-size **[[Page|pages]]** and physical memory into matching **[[Page Frame|page frames]]**. A per-process **[[Page Table]]** maps virtual page numbers (VPNs) to physical frame numbers (PFNs). The dominant scheme in every modern OS.

## The Core Idea

Fixed-size everything:

- Address space → array of **virtual pages** (typically 4 KB each).
- Physical memory → array of **page frames** (same size).
- Mapping is **arbitrary** — no contiguity required.

```
VPN    PFN
 0  →   3
 1  →   7
 2  →   5
 3  →   2
```

Page 0 of the process can sit at frame 3; page 1 at frame 7. Pages don't have to be in order, contiguous, or even all resident (some can be on disk — see [[Page Fault]]).

## Why Paging Wins

Compared to [[Segmentation]]:

| Property | Segmentation | Paging |
|---|---|---|
| Allocation unit | variable-size segment | fixed-size page |
| External fragmentation | yes (the killer) | none |
| Internal fragmentation | minimal | up to 1 page per allocation |
| Sharing | per-segment | per-page (very flexible) |
| Translation | base+bounds | page-table lookup |
| Hardware complexity | small | larger (MMU, [[TLB]]) |

Paging's **fixed sizes** eliminate external fragmentation entirely — the central allocation problem disappears.

## The Cost

Two new problems, fixed in the next two chapters:

- **Slow** — each access requires a page-table lookup, doubling memory traffic. Solved by the [[TLB]] (Ch 19).
- **Large tables** — 4 MB per process for a 32-bit VA with 4 KB pages. Solved by [[Multi-Level Page Table|multi-level]] / [[Inverted Page Table|inverted]] page tables (Ch 20).

## Translation in Pseudocode

```c
VPN      = VA >> PAGE_SHIFT;
offset   = VA &  PAGE_MASK;
PTE      = PageTable[VPN];

if (!PTE.Valid)        raise(SEGFAULT);
if (!PTE.AccessAllowed(op)) raise(PROT_FAULT);
if (!PTE.Present)      pageFaultHandler();

PA = (PTE.PFN << PAGE_SHIFT) | offset;
```

The MMU does this on **every** memory access. The [[TLB]] caches recent results to skip the `PageTable[VPN]` step.

## Page Size: Tradeoffs

- **Smaller pages** (e.g., 4 KB) — less internal fragmentation, but bigger page tables and more TLB misses for large data.
- **Larger pages** (e.g., 2 MB, 1 GB "huge pages") — fewer TLB misses, smaller tables, but more internal fragmentation.

Modern systems offer **multiple sizes simultaneously** (Linux: 4 KB default + 2 MB / 1 GB huge pages on demand).

## Where Paging Lives Today

- Every general-purpose OS (Linux, Windows, macOS, BSDs) uses paging.
- Hardware: x86_64 (4-level), ARM64 (4-level), RISC-V (Sv39, Sv48, Sv57). All converged on similar designs.
- Even VMs use **two levels** of paging — guest virtual to guest physical, then guest physical to host physical (Extended/Nested Page Tables).

## Related Notes

- [[Page]], [[Page Frame]] — units of virtual and physical memory.
- [[Page Table]], [[Page Table Entry]] — the translation data structure.
- [[TLB]] — the speed fix.
- [[Multi-Level Page Table]], [[Inverted Page Table]] — the space fixes.
- [[Page Fault]] — when a page isn't resident.
- [[Address Translation]] — paging is the dominant translation scheme.
- [[Memory Virtualization]] — paging is its primary mechanism.
- [[Ch 18 — Paging - Introduction]], [[Ch 19 — Paging - Faster Translations (TLBs)]], [[Ch 20 — Paging - Smaller Tables]].
