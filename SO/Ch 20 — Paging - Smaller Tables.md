---
title: Ch 20 — Paging - Smaller Tables
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
type: chapter
book: OSTEP
chapter: 20
pages: 229-242
---

# Ch 20 — Paging: Smaller Tables

> [!abstract] One-sentence summary
> A flat linear [[Page Table]] for a 32-bit address space already costs 4 MB per process — for 64-bit, hopeless. The chapter walks four ways to shrink it: **bigger pages** (cheap, brings internal fragmentation), **hybrid paging+segmentation** (Multics-style, brings back external fragmentation), **[[Multi-Level Page Table|multi-level page tables]]** (the universal modern answer), and **[[Inverted Page Table|inverted page tables]]** (one entry per physical frame, lookup via hash).

## Crux

> [!example] The Crux: How To Make Page Tables Smaller
> Simple array-based (linear) page tables are too big, taking up too much memory on typical systems. How can we make page tables smaller? What are the key ideas? What inefficiencies arise as a result of these new data structures?

## 20.1 Simple Solution — Bigger Pages

Quadruple the page size: 32-bit VA, 16 KB pages → 18-bit VPN → 2¹⁸ × 4 B = **1 MB** per page table (a 4× reduction matching the page-size growth).

> [!warning] Bigger pages → more [[Internal Fragmentation|internal fragmentation]]
> Allocating 9 KB now consumes 16 KB. Programs that use small pieces of many pages waste a lot. Most systems use **mixed sizes**: 4 KB default + 2 MB / 1 GB huge pages on demand.

## 20.2 Hybrid: Paging + Segmentation

Idea (Multics, Jack Dennis): **one page table per logical segment** instead of one per address space:

- Code segment → its own page table (small — only as many pages as code occupies).
- Heap → its own.
- Stack → its own.

Segment registers now hold the **physical address of the per-segment page table**, plus a bounds telling how many entries are valid.

> [!info] Translation
> ```
> SN  = top 2 bits of VA               # which segment?
> VPN = next bits of VA                 # offset within segment, in pages
> PTEAddr = SegBase[SN] + VPN * sizeof(PTE)
> ```

| Pro | Con |
|---|---|
| Per-segment tables sized to actual use → big memory savings | Inherits [[Segmentation]]'s assumption that the address space breaks into a few segments |
| Heap with sparse usage still wastes table entries within heap segment | Page tables are now **variable-sized** → external fragmentation in physical memory |

VAX/VMS used a variant. Modern systems chose multi-level instead.

## 20.3 Multi-Level Page Tables

The dominant modern approach. **Make the page table itself a tree**: chop the linear table into page-sized chunks, and have a small **page directory** that says which chunks actually have valid entries.

```
Linear (1 chunk, 4 MB)        Multi-level (page directory + needed chunks)
+----+----+----+----+----+    +-------+    +----+----+----+----+...
|    |    |    |    |    | →  |  PD   | →  | PT |    | PT |    |
+----+----+----+----+----+    +-------+    +----+    +----+
all entries allocated even      only          (only allocated for
if mostly invalid               page-       address-space regions
                                directory   actually in use)
                                always
                                resident
```

If the address space is sparse (most VPNs invalid), entire pages of the page table can be **omitted** — the page directory marks them invalid. Page table memory grows in proportion to **address space actually used**, not the maximum size.

### Translation: Two Lookups Per Miss

```c
PDIndex = (VPN & PD_MASK) >> PD_SHIFT;
PDE     = PageDirectory[PDIndex];
if (!PDE.Valid) raise(SEGFAULT);

PTIndex = VPN & PT_MASK;
PTE     = PageOf(PDE.PFN)[PTIndex];
if (!PTE.Valid) raise(SEGFAULT);

PA      = (PTE.PFN << SHIFT) | offset;
```

> [!warning] TLB miss now costs *two* memory accesses
> One to fetch the PDE, another to fetch the PTE. (In practice, the [[TLB]] absorbs almost all this — page-walk caches in modern CPUs further help.)

### x86_64 Has Four Levels

A 48-bit virtual address splits into four 9-bit indices plus a 12-bit offset:

```
[ 9 bits ][ 9 bits ][ 9 bits ][ 9 bits ][ 12-bit offset ]
   PML4      PDPT       PD         PT
```

Each level is itself a 4 KB page holding 512 8-byte entries. The whole tree is ~paged virtual memory for the page table; sparse address spaces yield small trees.

> [!info] Even more levels
> 5-level paging (LA57) extends VA to 57 bits — ARM64 has up to 4 or 5 depending on configuration; RISC-V Sv57 also has 5. Going deeper saves space at the cost of an additional memory access per page-table walk.

## 20.4 Inverted Page Tables

A different angle: instead of one table per process, have **one global table with one entry per physical frame**. Each entry records *which process and VPN* currently uses that frame.

| Pro | Con |
|---|---|
| Total table size = (number of frames) × (entry size). Independent of address-space size and number of processes. | Lookup is **search**, not index — needs a hash table to be fast. |
| Tiny on systems with huge VAs but moderate RAM | Sharing is awkward (one frame, multiple users). |

Used by PowerPC and some IA-64 designs. Linux supports an inverted-style scheme (page descriptors) but not as the primary translation table.

## 20.5 Swapping Page Tables Themselves

Even with multi-level structures, kernel page-table memory could exceed RAM on a heavily-loaded system. Some OSes (notably VAX/VMS) put **page tables in kernel virtual memory** so they themselves can be swapped. Heavy machinery; revisited in Ch 23.

## 20.6 Summary

> [!example] Crux answered
> Linear page tables waste memory on sparse address spaces. **Multi-level** tables (the modern winner) allocate page-table pages only where the address space is actually used. **Inverted** tables flip the indexing entirely. **Bigger pages** trade frag for size. The right choice depends on environment: tiny embedded systems may go inverted or huge-page-only; general-purpose OSes converged on multi-level.

## Aside: Time-Space Tradeoffs

> [!tip] The textbook tradeoff
> Multi-level tables are smaller but **slower** to walk (more memory accesses on a TLB miss). The TLB is what makes the slowness invisible most of the time. Whenever you build a data structure, ask: what's the *space* cost? what's the *access* cost? You can always trade one for the other.

## Aside: Multiple Page Sizes Today

x86_64, ARM64, RISC-V all support multiple page sizes simultaneously. Linux's transparent huge pages (THP) and explicit `MAP_HUGETLB` let applications request 2 MB or 1 GB pages where they help (databases, ML), while the bulk of allocations stay 4 KB.

## Related Notes

- [[Multi-Level Page Table]] — the dominant modern structure.
- [[Inverted Page Table]] — the alternative.
- [[Page Directory]] — top level of multi-level.
- [[Hybrid Paging-Segmentation]] — Multics/VAX style.
- [[Page Table]] — the abstract concept being optimized.
- [[Page Table Entry]] — what each leaf still holds.
- [[Sparse Address Space]] — what motivates multi-level structures.
- [[TLB]] — what hides the deeper walk costs.
- [[Internal Fragmentation]] — what bigger pages cost.
- [[Paging]] — the umbrella mechanism.
- Index: [[MOC - Memory Virtualization]].
