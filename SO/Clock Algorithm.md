---
title: Clock Algorithm
tags:
  - concept
  - ostep
  - memory
  - paging
  - policy
aliases:
  - Clock Replacement
  - Second-Chance Algorithm
type: concept
introduced_in: Ch 22
---

# Clock Algorithm

> [!abstract] Definition
> The **Clock algorithm** (Corbató, 1969) is the dominant practical approximation of [[LRU]]. Pages are arranged in a circular list; a **clock hand** sweeps around. On eviction, the hand passes pages with [[Reference Bit|use bit]] = 1 (clearing it as it goes) and evicts the first one with use bit = 0. Cheap enough for production; close enough to LRU in practice.

## Why Approximate LRU?

Strict LRU requires updating an ordered data structure on **every memory access** and scanning all PTEs at eviction time. Both are too expensive for systems with millions of pages. The Clock algorithm bounds eviction work to **at most one full sweep** of the page list while still evicting a "not recently used" page.

## The Mechanism

Hardware sets the **use bit** (`A` on x86, `R` on ARM) on every access. The OS clears the bit. The page list is circular; a hand points at the next eviction candidate.

```
                  ┌── page A (use=0)
                  ↓
        page H ← clock → page B (use=1)
                  ↓
                  page G (use=1)
                  ...
```

```c
PFN evict_clock(void) {
    while (true) {
        page = pages[hand];
        if (page->use == 0) {
            hand = (hand + 1) % N;
            return page->frame;          // evict this one
        }
        page->use = 0;                   // give a "second chance"
        hand = (hand + 1) % N;
    }
}
```

A page that was accessed since last sweep escapes (its bit gets cleared, but it gets one cycle to be touched again). A page not accessed gets evicted.

## Why It's Called "Second-Chance"

When the hand finds `use=1`, the algorithm doesn't evict immediately — it clears the bit and moves on. The page gets a "second chance": if accessed before the hand returns, it stays; otherwise next sweep finds it with `use=0` and evicts.

## Variants

### Clock with Dirty Bit

Prefer evicting **clean** pages (no disk I/O on eviction). The hand inspects (use, dirty):

| (use, dirty) | Action |
|---|---|
| (0, 0) | best candidate — evict immediately |
| (0, 1) | dirty but old — write back, then evict |
| (1, 0) | recent clean — clear use, advance |
| (1, 1) | recent dirty — clear use, advance |

The OS may make multiple passes, preferring "old + clean" first.

### Random Clock

Instead of a strict circular sweep, pick a random starting hand position each time. Resists pathological access patterns; cheaper to implement (no shared hand to lock under multicore).

### N-th Chance

Keep a small counter per page that increments each time the hand finds it with `use=0`. Evict when counter reaches N. Provides finer-grained "older = more evictable" ordering.

## Performance

OSTEP simulations show Clock is **slightly worse** than LRU but **vastly cheaper** to implement. On the 80/20 workload, Clock tracks LRU within a few percent. On scan-heavy workloads, both LRU and Clock collapse — the practical fix is **scan resistance** (ARC).

## Linux Today

Linux uses a **two-list LRU approximation** (active + inactive) that's clock-like in spirit but more elaborate:

- Pages start on the **inactive** list.
- Promoted to **active** on second access.
- Eviction always from the inactive list, preferring `use=0`.
- Periodic scans rebalance.

This combines Clock-style cheap accounting with ARC-style scan resistance.

## Related Notes

- [[LRU]] — what Clock approximates.
- [[Page Replacement]] — the umbrella.
- [[Reference Bit]] — the hardware support Clock requires.
- [[Page Table Entry]] — where the use and dirty bits live.
- [[Optimal Replacement]] — the unreachable bound.
- [[Belady's Anomaly]] — Clock can suffer it (no stack property).
- [[Ch 22 — Beyond Physical Memory - Policies]].
