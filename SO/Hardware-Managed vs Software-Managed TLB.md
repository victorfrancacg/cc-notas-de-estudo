---
title: Hardware-Managed vs Software-Managed TLB
tags:
  - concept
  - ostep
  - memory
  - cache
  - architecture
type: concept
introduced_in: Ch 19
---

# Hardware-Managed vs Software-Managed TLB

> [!abstract] Definition
> Two designs for handling [[TLB]] misses: **hardware-managed** (the MMU walks the [[Page Table]] itself, in silicon) versus **software-managed** (the MMU traps to the OS, which walks whatever data structure it wants and inserts the translation). The choice reflects the broader RISC vs CISC philosophy.

## Hardware-Managed (CISC: x86, x86_64, ARM*)

On a TLB miss, the MMU itself:

1. Consults a register that holds the page-table base (`CR3` on x86).
2. Walks the page-table tree (multi-level traversal).
3. Reads the leaf [[Page Table Entry|PTE]] from physical memory.
4. Inserts it into the TLB.
5. Retries the faulting instruction.

The OS is **not involved** in normal misses. It is involved only when the resulting PTE is invalid/missing (page fault) or when its policies need to update bits.

| Pros | Cons |
|---|---|
| Fast misses | Page-table format **fixed** by hardware |
| Simple OS code | Can't innovate on page-table data structure |
| Predictable performance | Hardware complexity grows with VA depth |

> [!info] x86_64 walks 4 levels, ARM64 walks up to 4
> The hardware traverses the [[Multi-Level Page Table|multi-level page table]] structure exactly as specified by the architecture. The OS just provides the tables in the prescribed format.

## Software-Managed (RISC: MIPS, SPARC, older Alpha)

On a TLB miss the MMU **traps** to the kernel:

```text
TLB miss → trap to OS handler →
   OS code: PTE = lookup(VPN);   // any data structure
            TLB_Insert(VPN, PTE.PFN);
            return-from-trap
hardware retries the instruction
```

| Pros | Cons |
|---|---|
| OS can use **any** page-table format (hash tables, multi-level, inverted, …) | Slower misses (trap overhead) |
| Lets researchers experiment | TLB-miss handler must avoid recursive misses |
| Hardware is simpler | Privileged instructions to update TLB (`TLBP`, `TLBR`, `TLBWI`, `TLBWR` on MIPS) |

> [!warning] Avoid recursive misses
> The miss handler is itself code in virtual memory. If accessing it causes another TLB miss, you have a chain. Solutions: keep the handler at unmapped (physical) addresses, or **wire** its translations permanently into the TLB.

## Why the Split Matches RISC vs CISC

- **CISC** philosophy — bake complex operations into hardware so binaries stay compact and fast for the common case.
- **RISC** philosophy — keep hardware simple; let software handle complexity. Provides flexibility — the OS can innovate on page-table representation freely.

Today most production CPUs are **hardware-managed** because TLB miss latency matters and the win for software-managed (data-structure flexibility) hasn't paid off in practice. MIPS and POWER kept software-managed designs longer than most.

## A Hybrid: Inverted Page Tables

The PowerPC architecture used a **hardware-managed [[Inverted Page Table|inverted hash table]]** for translations — an unusual middle ground. The OS still owns the data structure, but the hardware walks it on a miss.

## Related Notes

- [[TLB]] — what both schemes manage.
- [[Page Table]] — what the walker traverses.
- [[Multi-Level Page Table]] — the typical hardware-managed structure.
- [[Inverted Page Table]] — a structure software-managed TLBs can choose.
- [[MMU]] — where the TLB and walker live.
- [[Ch 19 — Paging - Faster Translations (TLBs)]].
