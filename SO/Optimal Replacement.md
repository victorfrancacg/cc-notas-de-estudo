---
title: Optimal Replacement
tags:
  - concept
  - ostep
  - memory
  - paging
  - policy
aliases:
  - MIN
  - Belady's Algorithm
  - Belady's Optimal
type: concept
introduced_in: Ch 22
---

# Optimal Replacement (MIN)

> [!abstract] Definition
> **Optimal replacement** (a.k.a. **MIN**, Belady 1966) evicts the [[Page|page]] that will be used **furthest in the future**. It minimizes the [[Page Fault|page-fault]] rate among all possible policies — a theoretical lower bound that requires future knowledge and is therefore **unimplementable** in production. Used as a benchmark to evaluate real policies.

## The Algorithm

```
On a page fault with full memory:
    for each resident page P:
        next_use[P] = next time P is referenced (∞ if never)
    evict argmax_P { next_use[P] }
```

The page kept the longest before its next use is the best one to discard, because every other resident page will be needed sooner.

## Why It's Optimal

**Intuition:** if any other page is referenced sooner, evicting it instead of the furthest-future page would cause a fault sooner. So evicting the furthest-future page postpones the next fault as much as possible. By induction over the reference stream, this minimizes total faults.

**Formal proof:** by exchange argument — any non-optimal eviction can be swapped with the optimal one without increasing the total fault count.

## Why It's Unusable

You'd need an oracle for the future reference stream. Real workloads are unpredictable; even if predictable in the abstract, instrumenting and analyzing them at runtime would dwarf the savings.

## Why We Care

> [!tip] Optimal as a benchmark
> "My new algorithm has 80% hit rate" is meaningless. "My new algorithm has 80% hit rate; optimal achieves 82%" tells you the algorithm is **near the ceiling** — further tuning has limited room.

OSTEP simulator results consistently show LRU within a few points of optimal on locality-heavy workloads. That's a strong endorsement of LRU.

## Related Notes

- [[Page Replacement]] — the umbrella.
- [[LRU]] — the practical close-to-optimal policy.
- [[Belady's Anomaly]] — the discovery that *non-optimal* policies can degrade with more memory.
- [[Clock Algorithm]] — the production-friendly LRU approximation.
- [[Ch 22 — Beyond Physical Memory - Policies]].
