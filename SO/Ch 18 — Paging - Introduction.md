---
title: Ch 18 — Paging - Introduction
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
type: chapter
book: OSTEP
chapter: 18
pages: 199-211
---

# Ch 18 — Paging: Introduction

> [!abstract] One-sentence summary
> Instead of managing variable-sized chunks ([[Segmentation]]), **paging** chops the [[Address Space]] into fixed-size **[[Page|pages]]** and physical memory into matching **[[Page Frame|frames]]**. The OS records each process's mapping in a per-process **[[Page Table]]**. This eliminates [[External Fragmentation|external fragmentation]] entirely — but creates two new problems: page-table lookups make every memory access slow, and the tables themselves are huge.

## Crux

> [!example] The Crux: How To Virtualize Memory With Pages
> How can we virtualize memory using pages, so as to avoid the problems of segmentation? What are the basic techniques? How do we make those techniques work well, with minimal space and time overheads?

## 18.1 A Simple Example

OSTEP uses a tiny example throughout: a **64-byte address space** with **16-byte pages** (so 4 virtual pages, VPN = 2 bits) and a **128-byte physical memory** (8 page frames, PFN = 3 bits).

```
Virtual Address Space (64 B)        Physical Memory (128 B)
 0  ┌──────────────┐            0  ┌─────────────────┐ reserved for OS
    │ page 0 of AS │           16  ├─────────────────┤
16  ├──────────────┤               │ (unused)        │
    │ page 1       │           32  ├─────────────────┤
32  ├──────────────┤               │ page 3 of AS    │
    │ page 2       │           48  ├─────────────────┤
48  ├──────────────┤               │ page 0 of AS    │  ← VPN 0 → PFN 3
    │ page 3       │           64  ├─────────────────┤
64  └──────────────┘               │ (unused)        │
                               80  ├─────────────────┤
                                   │ page 2 of AS    │  ← VPN 2 → PFN 5
                               96  ├─────────────────┤
                                   │ (unused)        │
                              112  ├─────────────────┤
                                   │ page 1 of AS    │  ← VPN 1 → PFN 7
                              128  └─────────────────┘
```

The mapping (the **page table**) for this process: `{VPN 0 → PFN 3, VPN 1 → PFN 7, VPN 2 → PFN 5, VPN 3 → PFN 2}`.

## Translation Mechanics

A virtual address splits into two parts:

```
6-bit virtual address:
+-----+-----+
| VPN | offset |
+-----+-----+
  2 bits  4 bits
```

To translate VA `21` (= `010101`):

1. Top 2 bits = `01` → VPN = 1.
2. Look up page table: `VPN 1 → PFN 7` (`111`).
3. Concatenate PFN with the bottom 4 bits (offset = `0101` = 5).
4. Physical address = `1110101` = 117.

The offset is **not translated** — it indexes within the page.

## 18.2 Where Are Page Tables Stored?

> [!warning] Page tables are huge
> 32-bit virtual address + 4 KB pages → 20-bit VPN → 2²⁰ ≈ **1 million PTEs** per process. At 4 bytes each, that's **4 MB per process**. 100 processes → 400 MB just for tables.

Too big for MMU registers (which can fit a few base/bounds pairs). Instead, **page tables live in OS-managed physical memory**, and the MMU is told where to find the current process's table via the **page-table base register (PTBR)**, e.g., x86's `CR3`.

## 18.3 What's in a Page Table Entry (PTE)?

A linear page table is **just an array indexed by VPN**. Each entry contains:

| Field | Meaning |
|---|---|
| **PFN** | physical frame number |
| **Valid bit** | is this VPN in use? Crucial for [[Sparse Address Space\|sparse address spaces]] |
| **Protection bits** | R / W / X — enforced by MMU on each access |
| **Present bit** | is this page resident in RAM, or swapped to disk? (Ch 21) |
| **Dirty bit** | has this page been modified since being brought in? |
| **Reference bit** | has this page been accessed? Used by [[Page Replacement\|replacement policies]] |
| **U/S bit** | user/supervisor — kernel-only pages |

Example x86 PTE layout:

```
31           12 11  9 8 7 6 5 4 3 2 1 0
+--------------+-----+-+-+-+-+-+-+-+-+-+
|     PFN      | ... |G|PAT|D|A|PCD|PWT|U/S|R/W|P|
+--------------+-----+-+-+-+-+-+-+-+-+-+
```

(`P` = present, `D` = dirty, `A` = accessed.)

## 18.4 Paging: Also Too Slow

Every memory reference now requires **two memory accesses**:

1. Fetch the PTE from the page table in physical memory (using PTBR + VPN × sizeof(PTE)).
2. Use the PFN to fetch the actual data.

```c
// Hardware translation (simplified)
VPN     = (VirtualAddress & VPN_MASK) >> SHIFT;
PTEAddr = PTBR + (VPN * sizeof(PTE));
PTE     = AccessMemory(PTEAddr);                  // ← memory access #1

if (!PTE.Valid)         RaiseException(SEGFAULT);
if (!CanAccess(PTE.ProtectBits)) RaiseException(PROT_FAULT);

offset   = VirtualAddress & OFFSET_MASK;
PhysAddr = (PTE.PFN << SHIFT) | offset;
Register = AccessMemory(PhysAddr);                // ← memory access #2
```

Best case: **2× slowdown**. Worse if multi-level page tables. Hence [[TLB|TLBs]] (Ch 19).

## 18.5 Memory Trace

A `for (i = 0; i < 1000; i++) array[i] = 0;` loop with paging makes a wild number of accesses:

- Every instruction fetch → 1 page-table access + 1 instruction fetch = **2 memory refs**.
- Every store → 1 page-table access + 1 store = **2 memory refs**.
- The four-instruction loop body costs **10 memory accesses per iteration** (4 instr fetches × 2 + 1 store × 2 = 10) instead of 5.

> [!warning] Without TLBs, paging is unusable
> A 2× slowdown across all memory traffic would have killed paging in the cradle. The next chapter exists for exactly this reason.

## 18.6 Summary

> [!example] Crux answered
> Paging eliminates [[External Fragmentation|external fragmentation]] (everything is a fixed-size page) and supports flexible address-space layouts. But two new problems emerge: **slow translation** (each VA requires an extra memory access) and **large page tables** (~4 MB per 32-bit process). The next two chapters fix these:
> - [[Ch 19 — Paging - Faster Translations (TLBs)|Ch 19]] — caches translations in the [[TLB]] to fix the speed problem.
> - [[Ch 20 — Paging - Smaller Tables|Ch 20]] — multi-level / inverted page tables to fix the space problem.

## Aside: Paging Predates You

> [!info] Atlas, Manchester, 1962
> Paging was invented for the Atlas computer at Manchester. The idea — fixed-size pages with a translation table — has been the foundation of every general-purpose OS since. Sixty-plus years and still the dominant scheme.

## Related Notes

- [[Paging]] — the standalone concept.
- [[Page]], [[Page Frame]] — virtual and physical units.
- [[Page Table]] — the translation table.
- [[Page Table Entry]] — a single mapping.
- [[Physical Address]] — what translation produces.
- [[Virtual Address]] — what translation consumes.
- [[MMU]] — performs the translation.
- [[External Fragmentation]] — what paging eliminates.
- [[Internal Fragmentation]] — paging's residual cost (last partial page).
- [[Segmentation]] — paging's predecessor and rival.
- Next: [[Ch 19 — Paging - Faster Translations (TLBs)]].
- Index: [[MOC - Memory Virtualization]].
