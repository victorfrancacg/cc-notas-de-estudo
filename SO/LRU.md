---
title: LRU
tags:
  - concept
  - ostep
  - memory
  - paging
  - policy
aliases:
  - Least Recently Used
  - LRU Replacement
type: concept
introduced_in: Ch 22
---

# LRU (Least Recently Used)

> [!abstract] Definition
> **LRU** evicts the [[Page|page]] that has been **least recently accessed**. It bets on [[Temporal Locality|temporal locality]] — pages used recently are likely to be used again soon. On locality-heavy workloads it approaches [[Optimal Replacement|optimal]] performance; on adversarial patterns (looping sequential) it can collapse.

## The Idea

Maintain an ordered list of resident pages, most-recent-first. On every access, move the page to the front. On eviction, take from the back.

```
After accesses: 0, 1, 2, 0, 1, 3, 0
LRU list (MRU first): [0, 3, 1, 2]
                       newest    oldest → evict next
```

## Stack Property

LRU has the **stack property**: for any reference stream, the cache of size N+1 always contains the same pages as the cache of size N (plus one more). Therefore LRU **cannot** suffer [[Belady's Anomaly]] — increasing cache size never makes hit rate worse.

FIFO and Random lack this property; LFU has a weaker form.

## Strengths

- **Matches optimal closely on locality-rich workloads.** OSTEP's 80/20 simulation: LRU near-optimal.
- **Stack property** — increasing memory always helps (or doesn't hurt).
- **Intuitive** — easy to reason about.

## Weaknesses

> [!warning] Looping-sequential is brutal
> Loop touching N+1 pages with N frames: every access evicts the page about to be re-referenced. **0% hit rate**. FIFO has the same problem; Random does much better.

> [!warning] Cold pages with high frequency stick around
> Pure LRU only tracks recency; LFU tracks frequency. A page accessed heavily early then cold forever stays in cache (under LFU) when it should leave. Hybrids (ARC, LRFU) blend both.

> [!warning] One-shot scans pollute the cache
> Reading a 1 TB file once would, under naive LRU, evict everything else. **Scan-resistant** variants (ARC, 2Q) only promote on second access.

## Why Perfect LRU Is Expensive

Implementation requires updating an ordered data structure **on every memory access**. Even with hardware-assisted timestamps:

- The MMU writes a timestamp into each PTE on access.
- On eviction, the OS scans **every PTE** to find the minimum timestamp.

For a million-page system, that scan is too slow. Hence approximations.

## Approximations

The dominant production approach is the [[Clock Algorithm]]:

- Hardware sets a [[Reference Bit|use bit]] on access.
- OS clears the bit periodically.
- On eviction, sweep until finding `use=0`.

This gives **approximate** LRU — the page evicted is "old enough not to have been touched since the last sweep" rather than strictly oldest. Cheap enough to use; close enough to LRU in practice.

## Sibling Policies

- **MRU** (Most Recently Used) — evict the *most* recently used. Useful when hot data is one-shot (e.g., scanning).
- **LFU** (Least Frequently Used) — evict by access count. Sticky cold pages problem.
- **ARC** (Adaptive Replacement Cache) — combines recency and frequency, scan-resistant. Used in ZFS and many DB systems.

## Related Notes

- [[Page Replacement]] — the umbrella.
- [[Optimal Replacement]] — the unreachable bound.
- [[Clock Algorithm]] — the practical LRU approximation.
- [[Belady's Anomaly]] — what LRU's stack property prevents.
- [[Reference Bit]] — hardware support for Clock.
- [[Temporal Locality]] — what LRU bets on.
- [[Ch 22 — Beyond Physical Memory - Policies]].
