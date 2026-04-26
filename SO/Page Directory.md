---
title: Page Directory
tags:
  - concept
  - ostep
  - memory
  - paging
aliases:
  - PDE
  - Page Directory Entry
type: concept
introduced_in: Ch 20
---

# Page Directory

> [!abstract] Definition
> The **page directory** is the **top level** of a [[Multi-Level Page Table]]. Each **page directory entry (PDE)** describes one page-sized chunk of the page table itself: if valid, it points to a page holding leaf [[Page Table Entry|PTEs]]; if invalid, that whole region of the address space is unmapped and **no memory is spent on a page-table page for it**.

## A Page Directory Entry

```
+---------+-------+-----+
| valid?  | PFN   | ... |
+---------+-------+-----+
```

| Field | Purpose |
|---|---|
| **valid** | does this PDE point to a page-table page that has at least one valid PTE? |
| **PFN** | physical frame containing that page-table page |
| **other bits** | sometimes: present, U/S, accessed — same as PTEs in deeper levels |

Note the **valid bit on a PDE means something different** from a PTE's valid bit. PDE.Valid = "the PT page exists and has at least one valid translation"; PTE.Valid = "this specific VPN is mapped".

## Use in Translation

```c
PDIndex = (VPN & PD_MASK) >> PD_SHIFT;
PDE     = PageDirectory[PDIndex];

if (!PDE.Valid) {
    // entire range of VPNs unmapped — no PT page allocated
    raise(SEGFAULT);
}

// PDE valid → fetch the leaf PTE
PTIndex = VPN & PT_MASK;
PTE     = PageOf(PDE.PFN)[PTIndex];
```

## Why It Saves Memory

For a sparse address space, most PDEs are invalid — and **the page-table pages they would point to are simply not allocated**. A linear page table would still allocate slots for every VPN, valid or not.

```
Address space layout: code at low, stack at high, gigabytes of unused middle.

Linear PT:        every VPN slot allocated         → MB
Multi-level PT:   PD allocated; PT pages only      → KB
                  for code-region and stack-region
```

## Modern Multi-Level Trees

x86_64 has 4 levels. The "page directory" terminology generalizes — each interior level's table is sometimes called PDPT, PD, PT (for the 2nd, 3rd, 4th levels). All have entries with the same fundamental form: valid bit + pointer to next-level table.

## Related Notes

- [[Multi-Level Page Table]] — the structure the page directory tops.
- [[Page Table]] — the umbrella concept.
- [[Page Table Entry]] — the leaf entries.
- [[Sparse Address Space]] — what page directories exploit.
- [[Paging]] — the mechanism.
- [[Ch 20 — Paging - Smaller Tables]].
