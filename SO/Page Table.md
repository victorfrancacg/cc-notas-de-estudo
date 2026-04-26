---
title: Page Table
tags:
  - concept
  - ostep
  - memory
  - paging
type: concept
introduced_in: Ch 18
detailed_in: Ch 20
---

# Page Table

> [!abstract] Definition
> A **page table** is the per-[[Process|process]] data structure that maps Virtual Page Numbers (VPNs) to Physical Frame Numbers (PFNs) under [[Paging]]. The [[MMU]] consults it on every memory reference (modulo [[TLB]] hits). It lives in physical memory; the kernel manages it; a register (e.g., x86 `CR3`) tells the MMU where it is for the currently-running process.

## Linear Page Table — The Simplest Form

A flat array indexed by VPN:

```
PageTable[VPN] = PTE
```

Each entry — a [[Page Table Entry|PTE]] — holds the PFN plus flags (valid, R/W/X, present, dirty, accessed). The MMU's translation:

```c
PTE = PageTable[VPN];
if (!PTE.Valid) raise(SEGFAULT);
PA  = (PTE.PFN << PAGE_SHIFT) | offset;
```

## Page Tables Are Big

For 32-bit virtual addresses with 4 KB pages:
- VPN = 20 bits → 2²⁰ entries.
- 4 bytes per entry → **4 MB per process**.
- 100 processes → **400 MB just for tables**.

For 64-bit addresses with 4 KB pages: a flat table is impossible (2⁵²-entries × 8 B = 32 PB). Hence **[[Multi-Level Page Table|multi-level]]** and **[[Inverted Page Table|inverted]]** page tables (Ch 20).

## Where Page Tables Live

In **physical memory**, allocated by the OS. The [[MMU]] is told the table's physical base address via:

| ISA | Register |
|---|---|
| x86 | `CR3` |
| ARM64 | `TTBR0`, `TTBR1` |
| RISC-V | `satp` |

On a [[Context Switch]], the OS updates the register to point at the new process's table — this is what swaps the address space.

## What's in a Page Table Entry — Recap

See [[Page Table Entry]]. Briefly:

| Field | Purpose |
|---|---|
| **PFN** | physical frame |
| **Valid** | is this VPN in use at all? |
| **R/W/X** | protection |
| **Present** | resident in RAM, or swapped out? |
| **Dirty** | modified since brought in |
| **Accessed** | touched recently — replacement hint |
| **U/S** | user vs kernel-only |

## Sparse Address Spaces

Modern address spaces are extremely [[Sparse Address Space|sparse]]: gigabytes of unused VA between code, heap, libraries, stack. A linear page table allocates an entry for every possible VPN — most marked invalid, but still occupying memory.

> [!tip] [[Multi-Level Page Table|Multi-level]] tables fix this
> Pages of the page table itself can be **omitted** if they would contain only invalid entries. Sparse address spaces consume tiny page tables.

## Per-Process

Each [[Process]] has its **own** page table. Two processes' VPN 0 map to *different* PFNs — the source of the [[Address Space|per-process address-space illusion]].

Sharing is achieved by mapping the **same** PFN at appropriate VPNs in two different processes' page tables (e.g., a shared library at the same physical pages, mounted at possibly different virtual addresses).

## Related Notes

- [[Page Table Entry]] — what each entry contains.
- [[Page]], [[Page Frame]] — what page tables map between.
- [[MMU]] — what consults the page table.
- [[TLB]] — what caches the result.
- [[Multi-Level Page Table]], [[Inverted Page Table]] — alternative structures.
- [[Paging]] — the umbrella mechanism.
- [[Context Switch]] — when the page-table base register is swapped.
- [[Ch 18 — Paging - Introduction]], [[Ch 20 — Paging - Smaller Tables]].
