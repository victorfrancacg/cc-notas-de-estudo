---
title: Belady's Anomaly
tags:
  - concept
  - ostep
  - memory
  - paging
  - policy
type: concept
introduced_in: Ch 22
---

# Belady's Anomaly

> [!abstract] Definition
> **Belady's Anomaly** (Belady, Nelson, Shedler, 1969) is the counterintuitive phenomenon where a **larger** cache produces **more** [[Page Fault|misses]] than a smaller one under certain access patterns. Specifically, [[Page Replacement|FIFO]] page replacement on the reference stream `1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5` has more faults with **4 frames** than with **3 frames**.

## The Famous Example

Reference stream: `1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`

| Cache size | Faults under FIFO |
|---|---|
| 3 | **9** |
| 4 | **10** |

More memory, *more* faults. Shocking the first time you see it.

## Why It Happens

FIFO doesn't respect access patterns — it just evicts the oldest resident page. With 4 frames the eviction order interacts with the reference pattern such that some pages get evicted right before they're re-referenced.

Specifically (with 4 frames):
- After loading 1,2,3,4 the cache holds {1,2,3,4}.
- Re-access to 1 and 2 hit, no eviction.
- Access to 5 evicts 1 (oldest); cache = {2,3,4,5}.
- Re-access to 1 misses; evict 2; cache = {3,4,5,1}.
- And so on.

With 3 frames, the eviction interactions happen to fall such that some hot pages stay resident longer.

## Which Policies Are Susceptible

| Policy | Belady-prone? |
|---|---|
| FIFO | yes |
| Random | yes |
| LFU | yes |
| **[[LRU]]** | no — has the **stack property** |
| **[[Optimal Replacement\|Optimal]]** | no |
| Clock (with dirty/use bits) | yes |

## The Stack Property

A policy has the **stack property** if for every reference stream, the cache of size N+1 is always a *superset* of the cache of size N. Stack-property policies cannot have Belady's anomaly: a fault with size N+1 implies a fault with size N (by superset relation), so faults monotonically decrease (or stay the same) as cache grows.

LRU has it. FIFO does not.

## Why It Matters

> [!tip] Why this matters in practice
> When sizing a cache (any kind — VM, web cache, database buffer pool), you generally assume "more memory = better hit rate". Belady's Anomaly is a counterexample showing that with the wrong policy, you can pay for more memory and get worse performance.
>
> Modern systems use stack-property algorithms (LRU, LRU-K, ARC) to avoid this.

## Related Notes

- [[Optimal Replacement]] — Belady also discovered MIN, the optimal policy.
- [[LRU]] — its stack property prevents the anomaly.
- [[Clock Algorithm]] — approximates LRU but doesn't strictly preserve the stack property.
- [[Page Replacement]] — the umbrella.
- [[Ch 22 — Beyond Physical Memory - Policies]].
