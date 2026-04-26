---
title: Ch 22 — Beyond Physical Memory - Policies
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
type: chapter
book: OSTEP
chapter: 22
pages: 273-290
---

# Ch 22 — Beyond Physical Memory: Policies

> [!abstract] One-sentence summary
> Under memory pressure, *which* page should the OS evict? **[[Optimal Replacement|Optimal]]** (Belady's MIN — evict the page used furthest in the future) is the unbeatable theoretical bound; real systems approximate it. **FIFO** and **Random** are simple but ignore history. **[[LRU|LRU]]** uses recency, matches optimal on locality-rich workloads, but is too expensive to implement perfectly — so production systems use **[[Clock Algorithm|approximations like Clock]]** built on a hardware-set **[[Reference Bit|use bit]]**.

## Crux

> [!example] The Crux: How To Decide Which Page To Evict
> When memory is full and a new page must be brought in, which resident page should be replaced to minimize the [[Page Fault|page-fault]] (cache-miss) rate?

## 22.1 Cache Management — [[AMAT|Average Memory Access Time]]

VM is a cache: RAM caches a subset of all pages. The cost of a typical memory reference is:

$$ AMAT = T_M + (P_{Miss} \cdot T_D) $$

where $T_M$ ≈ 100 ns (RAM), $T_D$ ≈ 10 ms (disk), $P_{Miss}$ = miss probability.

> [!warning] Even tiny miss rates dominate AMAT
> With $T_M = 100\text{ ns}$ and $T_D = 10\text{ ms}$:
> - $P_{Miss} = 0.001$ (99.9% hit) → AMAT = 100 ns + 10 µs ≈ **10 µs** (100× slower than RAM!)
> - $P_{Miss} = 0.1$ → AMAT ≈ **1 ms** (10000× slower)
>
> The disk-RAM gap is so wide that misses, not hits, set the runtime.

## 22.2 [[Optimal Replacement|Optimal Replacement (MIN)]]

> [!info] Belady, 1966
> Evict the page that will be used **furthest in the future**.

Yields the minimum possible miss rate. Unimplementable (requires future knowledge), but the **gold standard** every real algorithm is compared against.

> [!tip] Why optimal-as-baseline is useful
> "82% hit rate" is meaningless in isolation. "82% vs 85% optimal" tells you you're 3 points from the ceiling — close enough to stop tuning, or worth more work?

## 22.3 FIFO

Evict in arrival order — the oldest resident page goes first. **Simple**; ignores access patterns; can evict a hot page just because it's been resident longest. Vulnerable to **[[Belady's Anomaly]]**: more memory can give *fewer* hits.

## 22.4 Random

Evict any resident page uniformly at random. **Surprisingly competitive** — no pathological corner cases. Good baseline; some real systems use it where its simplicity outweighs LRU's tracking cost.

## 22.5 [[LRU|Least Recently Used]] (LRU)

Evict the page least-recently accessed. Bets on **[[Temporal Locality|temporal locality]]**: recent ⇒ likely soon. Has the **stack property** (cache of size N+1 always contains the cache of size N's pages) → never suffers Belady's anomaly.

> [!info] Sibling: LFU (Least Frequently Used)
> Evicts the page accessed least often. Less popular than LRU because of "old hot, now cold" pages that linger (high frequency lifts even cold pages to immortal status). Combined ARC-style policies handle this.

## 22.6 Workload Comparisons

OSTEP runs simulators on three reference patterns:

### No-locality (random over 100 pages)
All policies converge: cache size determines hit rate. Optimal slightly better than the rest. **Doesn't matter much which** policy you pick — locality is the carrier signal that policies exploit.

### 80/20 (80% of refs to 20% "hot" pages)
LRU clearly beats FIFO/Random — by holding hot pages, it tracks the working set. Optimal is still ahead.

### Looping-sequential (1, 2, ..., 50, 1, 2, ..., 50, ...)
FIFO and LRU **fail catastrophically**: 0% hit rate at cache size 49 (always evicts the page about to be reused). Random does much better — its randomness sometimes keeps the hot page. **Pathological case** for LRU; motivates **scan-resistant** modern algorithms.

## 22.7 Implementing LRU Is Expensive

Perfect LRU requires **updating a data structure on every access** — every load and store. Even with hardware help (timestamps in PTEs, per-frame access-time fields), **scanning a million-PTE table** to find the smallest timestamp is too slow.

> [!example] Crux: How to approximate LRU
> Given that perfect LRU is too expensive at scale, how can we approximate its behavior with cheap hardware support?

## 22.8 [[Clock Algorithm|Approximating LRU — the Clock Algorithm]]

Use a single hardware-set [[Reference Bit|reference bit]] (a.k.a. **use bit**) per page:

1. Hardware sets the bit on every access.
2. The OS clears it as part of replacement.

Arrange all pages in a **circular list**. Maintain a **clock hand**:

```
       page A [use=0]
            ↓
    page H ← clock → page B [use=1]
            ↓
       page G [use=0]
       (etc.)
```

To evict:
- Look at the page under the hand.
- If `use=1`, clear it, advance the hand. (Spared this round; comes back next.)
- If `use=0`, evict it.

Pages accessed since the last sweep escape; pages not accessed get evicted. **Approximates LRU** at constant per-eviction cost. Inexpensive enough to use in real kernels.

## 22.9 Considering Dirty Pages

Add the **dirty bit** (already in PTEs):

- **Clean** evictions: just discard (page can be re-fetched from disk if reused).
- **Dirty** evictions: must write back to disk first — expensive.

Modified clock variants prefer **clean** victims over dirty ones, accepting slight LRU deviation for fewer disk writes.

## 22.10 Other VM Policies

Beyond replacement, the OS also chooses:

- **[[Demand Paging|Page selection]]** — when to bring pages in (default: on demand). Pre-fetching brings consecutive pages early when patterns suggest it'll pay off.
- **Clustering** — write multiple dirty pages in one disk pass to amortize seek cost. Critical for HDDs; less so for SSDs.

## 22.11 [[Thrashing]]

When the working sets of all running processes don't fit in RAM, the system spends all time paging. Symptoms: very high disk I/O, CPU near idle, system unresponsive.

**Mitigations:**
- **Working-set model** ([[Working Set|see note]]) — each process's hot pages must fit; otherwise admit fewer processes.
- **Admission control** — refuse to run new processes when memory is tight.
- **Out-of-memory killer** — Linux's pragmatic last resort: pick a memory-hogging process and kill it. Crude, but better than total system meltdown.

## 22.12 Summary

> [!example] Crux answered
> **No replacement policy is universally best.** Optimal sets the ceiling; LRU and its approximations (Clock + variants) are the modern standard. Modern systems also add **scan resistance** (e.g., ARC) to dodge the looping-sequential cliff. With cheap RAM, replacement quality matters less — but with SSDs reviving the IO speed gap, smart algorithms are getting renewed attention.

## Aside: Locality

> [!info] Two flavors
> - **[[Spatial Locality]]** — accessed `x` ⇒ likely soon access `x±1`.
> - **[[Temporal Locality]]** — accessed `x` ⇒ likely soon re-access `x`.
>
> Replacement policies bet on temporal locality (LRU = "recent ⇒ soon"). Without locality, no policy beats random by much.

## Aside: Stack Property

A policy has the **stack property** if for every reference stream, the contents of a cache of size N+1 are a *superset* of the contents of size N. Implies no Belady's anomaly. LRU has it; FIFO does not.

## Related Notes

- [[Page Replacement]] — the umbrella concept.
- [[Optimal Replacement]] — Belady's MIN, the theoretical bound.
- [[LRU]] — the workhorse history-based policy.
- [[Clock Algorithm]] — the practical LRU approximation.
- [[Belady's Anomaly]] — why FIFO is treacherous.
- [[AMAT]] — the metric the chapter optimizes.
- [[Reference Bit]] — the hardware support Clock relies on.
- [[Working Set]] — the resident-set model behind admission control.
- [[Thrashing]] — what happens when the working set doesn't fit.
- [[Spatial Locality]], [[Temporal Locality]] — why caching works.
- [[Page Fault]] — what each "miss" really costs.
- [[Page Table Entry]] — where reference and dirty bits live.
- [[Ch 21 — Beyond Physical Memory - Mechanisms]].
- Index: [[MOC - Memory Virtualization]].
