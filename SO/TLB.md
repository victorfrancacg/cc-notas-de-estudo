---
title: TLB
tags:
  - concept
  - ostep
  - memory
  - cache
  - hardware
aliases:
  - Translation Lookaside Buffer
  - Address-Translation Cache
type: concept
introduced_in: Ch 18
detailed_in: Ch 19
---

# TLB (Translation Lookaside Buffer)

> [!abstract] Definition
> The **TLB** is a small, fully-associative hardware cache inside the [[MMU]] that holds recent virtual-to-physical translations. On every memory access the [[MMU]] checks the TLB first; a **hit** translates in essentially zero cycles. A **miss** falls through to a full [[Page Table]] walk. The TLB is what makes [[Paging|paging]] fast enough to use.

## Why It Exists

Paging requires a [[Page Table]] lookup on every memory access. Without help, every load/store/instruction-fetch costs one extra memory reference (or several, for [[Multi-Level Page Table|multi-level tables]]). That doubles or triples the cost of every operation — unusable.

The TLB sits between the CPU and the page-table walker, holding maybe 64 to 1024 of the most recently used translations. Most programs touch a small working set repeatedly, so most accesses hit the TLB.

## Anatomy of a TLB Entry

```
+-----+-----+-------+----------+-------+-------+
| VPN | PFN | valid | protect  | dirty | ASID  |
+-----+-----+-------+----------+-------+-------+
```

| Field | Purpose |
|---|---|
| **VPN** | virtual page number — what we look up by |
| **PFN** | physical frame number — what we get back |
| **valid** | does this slot hold a real translation right now? |
| **protect** | R/W/X permissions, copied from the [[Page Table Entry\|PTE]] |
| **dirty** | has the page been written to? |
| **ASID** | which address space this translation belongs to (see [[ASID]]) |

The TLB is **fully associative**: a translation can live in any slot, and lookup compares against every slot in parallel.

## TLB Hits and Misses

> [!info] TLB hit
> The translation is in the TLB. ~1 cycle. The MMU forms the physical address and the load/store proceeds.

> [!warning] TLB miss
> Not in the TLB. The MMU (or the OS, on software-managed TLBs) walks the [[Page Table]] to find the translation. Cost: one extra memory access for a flat table; two for two-level; four for four-level x86_64. Hundreds of cycles for an L3-miss page-table walk.

The TLB miss rate is one of the **dominant determinants of program performance** for memory-heavy workloads. Database systems, large-graph algorithms, and ML inference all care intensely about TLB coverage.

## Hit Rate via Locality

```c
for (int i = 0; i < N; i++) sum += a[i];
```

For an int array on 4 KB pages: 1024 ints per page. The first access to each page misses; the next 1023 hit. Hit rate ≈ 99.9%. This is **[[Spatial Locality]]** — adjacent VAs hit the same TLB entry.

If the loop iterates twice, the second iteration is ~100% hit (TLB warm). This is **[[Temporal Locality]]**.

## TLB Coverage

> [!warning] Working set vs TLB capacity
> A 64-entry TLB with 4 KB pages covers only 256 KB of memory. Random-access workloads on data >256 KB miss almost every access — *RAM isn't always RAM* (Culler's Law).

Mitigations:
- **Huge pages** — 2 MB or 1 GB pages multiply coverage by 512× / 262144×.
- **Multi-level TLBs** — a small fast L1 TLB backed by a larger slower L2 TLB (modern x86, ARM).

## Context Switches Are Hard

A TLB entry is valid only for the process whose page table it came from. After a [[Context Switch]], stale entries are dangerous:

- **Flush all** on switch — simple, expensive (cold TLB).
- **Tag entries with [[ASID|ASIDs]]** — entries from multiple processes coexist; mismatch on lookup is treated as a miss. Modern hardware (PCID on x86, ASID on ARM).

## Hardware-Managed vs Software-Managed

See [[Hardware-Managed vs Software-Managed TLB]]. Briefly:

- **HW-managed** (x86): the MMU walks the page table itself on a miss.
- **SW-managed** (MIPS, RISC): a TLB miss traps to the OS; the OS walks any data structure it likes and inserts the translation.

## Replacement

When the TLB is full, an incoming translation evicts an existing one. Real hardware uses **approximate LRU** (e.g., NRU, clock-like). Random eviction is sometimes used for resistance to pathological loops.

## Related Notes

- [[Page Table]] — what the TLB caches.
- [[Page Table Entry]] — the row that gets cached.
- [[MMU]] — where the TLB lives.
- [[ASID]] — multi-process tagging.
- [[Spatial Locality]], [[Temporal Locality]] — why TLB hit rates are high.
- [[Hardware-Managed vs Software-Managed TLB]] — miss-handling alternatives.
- [[Context Switch]] — when the TLB becomes a problem.
- [[Paging]] — the scheme TLBs accelerate.
- [[Ch 19 — Paging - Faster Translations (TLBs)]].
