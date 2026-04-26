---
title: AMAT
tags:
  - concept
  - ostep
  - memory
  - cache
  - performance
aliases:
  - Average Memory Access Time
type: concept
introduced_in: Ch 22
---

# AMAT (Average Memory Access Time)

> [!abstract] Definition
> **AMAT** is the expected cost of a memory reference once cache hits and misses are factored in. For [[Memory Virtualization|virtual memory]]: $AMAT = T_M + (P_{Miss} \cdot T_D)$ — the cost of always paying for RAM, plus the probability-weighted disk-access cost on a miss. The metric replacement policies optimize.

## Formula

$$ AMAT = T_M + (P_{Miss} \cdot T_D) $$

| Symbol | Meaning |
|---|---|
| $T_M$ | RAM access time (~100 ns on modern hardware) |
| $T_D$ | Disk (or swap) access time (~10 ms HDD, ~100 µs SSD) |
| $P_{Miss}$ | Page-miss probability (page-fault rate) |

The first term is paid every access; the second is the *expected* extra cost of a fault.

## Why Disk Latency Dominates

> [!warning] Even tiny miss rates wreck performance
> With $T_M = 100\text{ ns}$ and $T_D = 10\text{ ms}$:
> - $P_{Miss} = 0.001$ → AMAT = 100 ns + 10 µs ≈ **10 µs** (~100× slower than RAM)
> - $P_{Miss} = 0.01$ → AMAT ≈ 100 µs (~1000× slower)
> - $P_{Miss} = 0.10$ → AMAT ≈ 1 ms (~10000× slower)

A 99% hit rate sounds great until you compute AMAT.

## The Three C's of Misses

A useful decomposition (Hill, 1987):

- **Compulsory** (cold-start) — first reference to a page; unavoidable.
- **Capacity** — cache too small to hold the working set.
- **Conflict** — set-associative caches limit where a page can live (doesn't apply to fully-associative VM, but does to L1/L2).

Replacement policy can only reduce **capacity** misses. Demand prefetching tackles compulsory ones.

## CPU Cache Analog

The same formula generalizes to L1/L2/L3 caches with their own $T$ and $P_{Miss}$ values. Modern computer-architecture analyses chain them: AMAT_L1 = T_L1 + miss_L1 × AMAT_L2, etc.

## Why Replacement Algorithms Matter

A 1 percentage-point reduction in $P_{Miss}$ saves *tens of microseconds per access* (when $T_D \approx 10$ ms). For a program that does billions of memory accesses, that's hours.

Hence the obsession with [[LRU|LRU]], [[Clock Algorithm|Clock]], and modern variants like ARC. Each fault avoided is real, measurable performance.

## Related Notes

- [[Page Replacement]] — the policy AMAT motivates.
- [[Page Fault]] — the costly event AMAT measures.
- [[Memory Hierarchy]] — AMAT applies at every layer.
- [[Spatial Locality]], [[Temporal Locality]] — what makes $P_{Miss}$ low.
- [[Ch 22 — Beyond Physical Memory - Policies]].
