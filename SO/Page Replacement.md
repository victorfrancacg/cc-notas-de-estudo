---
title: Page Replacement
tags:
  - concept
  - ostep
  - memory
  - paging
  - policy
aliases:
  - Page Replacement Policy
  - Eviction Policy
type: concept
introduced_in: Ch 22
---

# Page Replacement

> [!abstract] Definition
> **Page replacement** is the OS policy that selects which resident [[Page|page]] to **evict** from physical memory when a new page must be brought in and no free [[Page Frame|frames]] are available. The choice has enormous performance impact because evicting the wrong page can multiply program runtime by 10⁴×.

## Why It Matters So Much

The cost of a [[Page Fault|page fault]] is dominated by disk access (~10 ms for HDD, ~100 µs for SSD), while RAM access is ~100 ns. Each avoided fault saves four to five orders of magnitude. So replacement quality scales linearly with memory-pressure performance.

[[AMAT|Average Memory Access Time]]:

$$ AMAT = T_M + (P_{Miss} \cdot T_D) $$

A 1 percentage-point increase in miss rate is worth tens of microseconds *per access*.

## The Spectrum of Policies

Ordered roughly from theoretical to practical:

| Policy | Idea | Verdict |
|---|---|---|
| **[[Optimal Replacement\|Optimal / MIN]]** | evict page used **furthest** in the future | unimplementable; theoretical bound |
| **FIFO** | oldest in, oldest out | simple, vulnerable to [[Belady's Anomaly]] |
| **Random** | random victim | surprisingly competitive |
| **[[LRU\|LRU]]** | least-recently-used | matches optimal on locality-heavy workloads, expensive to implement perfectly |
| **LFU** | least-frequently-used | rarely used alone (cold pages with old high counts) |
| **[[Clock Algorithm\|Clock]]** | approximate LRU using a [[Reference Bit\|use bit]] | the **dominant practical** approach |
| **ARC** (Modified Clock + scan resistance) | adapt to changing workload | modern caches (databases, file caches) |

## Approximating LRU in Practice

Maintaining a strict access-time per page is too expensive. Real systems:

1. Use a hardware-set [[Reference Bit|use bit]] per page (set on every access by the MMU).
2. Periodically clear the bits (the OS).
3. On eviction, prefer pages with `use=0`.

The [[Clock Algorithm]] is the canonical realization; modifications layer on dirty-page preference, scan resistance, and per-process resident-set limits.

## Local vs Global Policies

- **Global** — eviction picks from all pages of all processes. Maximizes overall miss rate but lets a "memory hog" steal from everyone. Linux mostly does this.
- **Local** — each process has its own page pool (resident set); only its pages are eligible for eviction. Older systems (VAX/VMS) used **segmented FIFO** with a per-process resident set.

Global is simpler; local is fairer.

## Pathological Cases

> [!warning] Looping-sequential
> A loop walking N+1 pages with N frames available → 0% hit rate under FIFO and LRU. Random does much better.

> [!warning] Scans of huge files
> A backup process reading a 1 TB file would normally evict everything else. Modern policies (ARC, 2Q) **resist scans** by promoting pages only on second access.

## Interaction with Other Subsystems

- **Dirty bit** — clean pages cost no I/O to evict (just discard). Clock variants prefer clean.
- **File cache** — file-backed pages and anonymous pages compete for memory. Linux balances via *swappiness*.
- **NUMA** — on multi-socket systems, local-node evictions are cheaper.
- **Compression** (zswap, zram) — compress evictees in RAM instead of writing to disk.

## Related Notes

- [[Optimal Replacement]] — the unreachable bound.
- [[LRU]] — the dominant heuristic.
- [[Clock Algorithm]] — the practical LRU approximation.
- [[Belady's Anomaly]] — pitfall of FIFO.
- [[AMAT]] — the metric.
- [[Reference Bit]] — hardware support.
- [[Working Set]] — model for admission control.
- [[Thrashing]] — what happens when no policy can save you.
- [[Page Fault]] — the event replacement is fighting.
- [[Ch 22 — Beyond Physical Memory - Policies]].
