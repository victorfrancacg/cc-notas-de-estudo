---
title: ASID
tags:
  - concept
  - ostep
  - memory
  - cache
aliases:
  - Address Space Identifier
  - PCID
  - Process-Context Identifier
type: concept
introduced_in: Ch 19
---

# ASID (Address Space Identifier)

> [!abstract] Definition
> An **ASID** is a small integer (typically 8 to 12 bits) that the OS assigns to each [[Process|process]] and the hardware stores in each [[TLB]] entry. The MMU compares the running process's ASID against the entry's ASID on every lookup; mismatches are treated as misses. ASIDs let the TLB hold translations from many processes simultaneously, dodging full TLB flushes on every [[Context Switch|context switch]].

## The Problem ASIDs Solve

Without ASIDs, a TLB entry valid for process P1 is **dangerous** if P2 starts running and shares no address space with P1:

```
TLB after P1 ran:
  VPN 10 → PFN 100  (P1's mapping)

P2 starts; it has VPN 10 → PFN 170, but the TLB still says PFN 100.
P2 reads VPN 10 and gets PFN 100 — process isolation is broken.
```

The naive fix: **flush the entire TLB on every context switch**. Correct, but slow — the new process runs with a cold TLB.

## How ASIDs Fix It

Tag each TLB entry with the ASID of the process it belongs to:

| VPN | PFN | valid | ASID |
|---|---|---|---|
| 10 | 100 | 1 | 1 (P1) |
| 10 | 170 | 1 | 2 (P2) |

The MMU stores the **current process's ASID** in a special register (set by the OS on every context switch). Translation now requires a match on both VPN *and* ASID. Stale entries from other processes are simply ignored — no flush needed.

## ASID Recycling

ASIDs are bounded (often 8 bits → 256). With more processes than IDs, the OS reuses old ASIDs:

1. Pick a recycled ASID for a new process.
2. **Flush all TLB entries with that ASID** (so old mappings don't haunt the new process).

So flushes still happen — just rarely, and per-ASID rather than wholesale.

## Modern Names

| ISA | Name | Bits |
|---|---|---|
| ARM | ASID | 8 (older) or 16 (modern) |
| x86 | PCID (Process-Context Identifier) | 12 |
| MIPS | ASID | 8 |
| RISC-V | ASID | varies (configurable) |

x86's PCID is relatively recent (Westmere, 2010); for years before, x86 paid the full-flush cost.

## Global Translations

Some translations are valid in **every** address space — the kernel's own pages, mapped into every process's upper half (so syscalls don't cost a full address-space switch). These TLB entries set a **global bit**, telling the MMU to ignore the ASID. They survive ASID changes and are flushed only explicitly.

## Why Not Just More ASID Bits

Wider ASIDs mean wider TLB entries, more silicon, more comparators. The bit-budget is small in associative hardware. 8–12 bits is the historical sweet spot.

## Related Notes

- [[TLB]] — what ASIDs tag.
- [[Context Switch]] — the operation ASIDs accelerate.
- [[Process]] — what each ASID identifies.
- [[Page Table]] — distinct per process; ASIDs avoid the TLB-flush cost when switching.
- [[Ch 19 — Paging - Faster Translations (TLBs)]].
