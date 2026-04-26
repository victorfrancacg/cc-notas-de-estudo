---
title: Allocation Strategy
tags:
  - concept
  - ostep
  - memory
  - allocation
aliases:
  - Allocator Policies
  - Fit Policy
type: concept
introduced_in: Ch 17
---

# Allocation Strategy

> [!abstract] Definition
> An **allocation strategy** is the policy a [[Free-Space Management|free-space manager]] uses to choose **which** free chunk to satisfy a request from. Mechanism (splitting, coalescing, headers) is the same across allocators; policy decides where to cut.

## The Classic Four

### Best Fit
Scan the entire free list; pick the **smallest** chunk that's ≥ the request.

| Pro | Con |
|---|---|
| Minimizes leftover per allocation | Full-list scan; produces many tiny remainders |

### Worst Fit
Scan; pick the **largest** chunk. Leftover stays big.

| Pro | Con |
|---|---|
| Tries to preserve large free regions | Empirically fragments *worse* than best-fit |

### First Fit
Return the **first** chunk that fits.

| Pro | Con |
|---|---|
| Fast — no full scan | Concentrates small leftovers at front of list |

Remedy: keep the free list **address-ordered**. Coalescing is cheap (adjacency = address arithmetic), and small leftovers spread instead of pooling.

### Next Fit
Like first-fit, but maintain a **rover pointer** that resumes search where the last alloc landed.

| Pro | Con |
|---|---|
| Fast; spreads leftovers | Performance ≈ first-fit |

## Worked Example

Free list: `10 → 30 → 20`. Request = 15.

| Policy | Picks | Resulting list |
|---|---|---|
| Best-fit | 20 | 10 → 30 → 5 |
| Worst-fit | 30 | 10 → 15 → 20 |
| First-fit | 30 | 10 → 15 → 20 |
| Next-fit | depends on rover | varies |

## When Each Wins

> [!tip] Workload-dependent — no clean winner
> - **Best-fit** wins when allocations are mixed sizes and you need every byte.
> - **First-fit** wins when speed matters more than space efficiency.
> - **Worst-fit** loses on most realistic workloads — historical curiosity.
> - **Next-fit** is a small first-fit refinement.

## Beyond the Four

- **[[Segregated List|Segregated lists]]** — bypass the choice entirely with size-class pools. Massive speedup for popular sizes.
- **[[Buddy Allocator]]** — cheap coalescing via power-of-two structure.
- **Trees (RB, splay, partially-ordered)** — `O(log n)` best-fit instead of `O(n)`.

Real allocators combine techniques: glibc malloc has small-bin (segregated), large-bin (sorted by size), tcache (per-thread small-bin cache), `mmap` for huge requests.

## Related Notes

- [[Free-Space Management]] — the umbrella concept.
- [[Splitting and Coalescing]] — what every policy depends on.
- [[Buddy Allocator]], [[Slab Allocator]] — alternative structures.
- [[External Fragmentation]] — the enemy.
- [[Ch 17 — Free-Space Management]].
