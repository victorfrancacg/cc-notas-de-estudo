---
title: Multi-Level Page Table
tags:
  - concept
  - ostep
  - memory
  - paging
aliases:
  - Hierarchical Page Table
  - Two-Level Page Table
  - Four-Level Page Table
type: concept
introduced_in: Ch 20
---

# Multi-Level Page Table

> [!abstract] Definition
> A **multi-level page table** is a [[Page Table]] organized as a **tree** of fixed-size pages: a top-level **[[Page Directory|page directory]]** points to second-level page-table pages, which (in deeper trees) point to third-level pages, and so on. Pages of the table that would contain only invalid entries are **omitted** — so memory consumed grows with address space *actually used*, not the maximum size. The dominant modern structure.

## The Idea

A linear [[Page Table]] for a 32-bit address space with 4 KB pages has 1 M entries × 4 B = 4 MB **per process**. Most of the address space is unused. With a multi-level table, **only the populated regions cost page-table memory**.

```
Linear PT                       Multi-level PT (2 levels)

Direct array of PTEs            +---- Page Directory ----+
indexed by VPN                  | PDE 0  → page of PT    |
                                | PDE 1  → invalid       |
| PFN | PFN | ... | PFN |       | ...                    |
| PFN | PFN | ... | PFN |       | PDE N  → page of PT    |
| PFN | PFN | ... | PFN |       +-------------------------+
                                       |               |
                                       v               v
                                +- page of PT -+ +- page of PT -+
                                | PTE entries | | PTE entries  |
                                +-------------+ +--------------+
   4 MB for everything,           Page directory always resident
   even invalid entries           (small); page-table pages
                                  allocated only as needed.
```

## Translation

Split the VPN into pieces — one per level:

```
VPN = [ PD index | PT index ]      (2-level)
VPN = [ PML4 | PDPT | PD | PT ]    (x86_64, 4-level)
```

```c
// 2-level translation
PDIndex = (VPN & PD_MASK) >> PD_SHIFT;
PDE     = PageDirectory[PDIndex];
if (!PDE.Valid) raise(SEGFAULT);

PTIndex = VPN & PT_MASK;
PTE     = PageOf(PDE.PFN)[PTIndex];
if (!PTE.Valid) raise(SEGFAULT);

PA      = (PTE.PFN << SHIFT) | offset;
```

A TLB miss now costs **N memory accesses** for an N-level tree. The [[TLB]] hides this in the common case.

## x86_64: Four Levels

48-bit VA with 4 KB pages:

```
[ 9 bits PML4 | 9 bits PDPT | 9 bits PD | 9 bits PT | 12 bits offset ]
```

Each level is a 4 KB page holding 512 × 8-byte entries. The entire walk on a TLB miss is 4 sequential memory accesses (mitigated by page-walk caches in modern CPUs).

## Why It Works on Sparse Address Spaces

Modern address spaces are extremely [[Sparse Address Space|sparse]] — gigabytes of unused VA between code/heap/libraries/stack. With a flat table, every potential VPN consumes a slot. With a multi-level tree, **entire pages of the page table are simply absent** when the corresponding range of VPNs is unmapped.

A typical Linux process with maybe 50 MB of mapped memory needs only a few KB of page-table memory — instead of 4 MB.

## Trade-Offs

| Pro | Con |
|---|---|
| Memory ∝ address space used, not max | More memory accesses per TLB miss |
| Each piece fits in one page (clean management) | More complex hardware/software |
| Supports huge sparse address spaces | Tree depth grows for wider VAs |

## "More Than Two Levels"

If even the page directory grows too big (because each PDE must fit in a page), split it into two levels — yielding a **3-level** or **4-level** tree. This is exactly what x86_64's 4 levels does: each level's index is 9 bits so each level's table fits in one 4 KB page (2⁹ × 8 B = 4 KB).

## Related Notes

- [[Page Table]] — the umbrella concept.
- [[Page Directory]] — the top level.
- [[Page Table Entry]] — the leaf.
- [[Inverted Page Table]] — the alternative scheme.
- [[Sparse Address Space]] — what makes multi-level worthwhile.
- [[Paging]] — the mechanism.
- [[TLB]] — hides the multi-level walk cost.
- [[Hybrid Paging-Segmentation]] — an older middle-ground attempt.
- [[Ch 20 — Paging - Smaller Tables]].
