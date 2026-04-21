---
title: CFS - Completely Fair Scheduler
tags:
  - concept
  - ostep
  - scheduling
  - algorithm
  - linux
aliases:
  - CFS
  - Completely Fair Scheduler
  - Linux CFS
  - vruntime
type: concept
introduced_in: Ch 9
---

# CFS — Completely Fair Scheduler

> [!abstract] Definition
> **CFS** is the default Linux scheduler since kernel 2.6.23 (2007). It implements [[Proportional Share Scheduling|fair-share scheduling]] via a simple accumulating counter per process (**virtual runtime / vruntime**), scales to thousands of processes via a **red-black tree**, and supports priorities via classic UNIX **nice** values.

Designed by Ingo Molnár. "Completely fair" is aspirational: in the simple case, every runnable process gets an equal share of CPU over any small window.

## Core Idea — Virtual Runtime

- Every process has a `vruntime`.
- As it runs, `vruntime` increases.
- The scheduler always picks the runnable process with the **lowest `vruntime`**.

In the simplest case, `vruntime` grows at real-time rate — so pick-lowest means "whoever has had the least CPU runs next". That's fair sharing.

## Key Parameters

### `sched_latency` (default ~48 ms)

A target window in which every runnable process should get its share. With *n* runnable processes, per-process slice = `sched_latency / n`.

### `min_granularity` (default ~6 ms)

A floor on the per-process slice. Without it, `sched_latency / n` would become absurdly small when many processes exist, and context-switching overhead would dominate.

For example, with 100 processes, naïve division gives 0.48 ms per process — way too small. `min_granularity` clamps this to 6 ms, at the cost of slightly stretching `sched_latency` to `6 ms × 100 = 600 ms` in the worst case.

## Priorities via `nice`

UNIX `nice` levels: **−20 (highest priority)** to **+19 (lowest)**. Default 0.

CFS maps nice → **weight**:

| Nice | Weight |
|---:|---:|
| −20 | 88761 |
| −5 | 3121 |
| 0 | 1024 |
| 5 | 335 |
| 10 | 110 |
| 19 | 15 |

Higher weight → larger time slice, and `vruntime` accumulates **slower** (so the process is picked more often).

### Slice Formula

$$
\text{slice}_k = \frac{\text{weight}_k}{\sum_i \text{weight}_i} \cdot \text{sched\_latency}
$$

### vruntime Formula

$$
\text{vruntime}_i = \text{vruntime}_i + \frac{\text{weight}_0}{\text{weight}_i} \cdot \text{runtime}_i
$$

Where `weight_0 = 1024` (nice 0's weight). Higher-weight processes have vruntime advance slower → chosen more often.

> [!tip] Ratios are preserved across nice
> If A has nice −5 (weight 3121) and B has nice 0 (weight 1024), the ratio is ~3:1. If instead A had nice 5 and B nice 10, the ratio would be almost identical. CFS preserves ratios across the full range.

## Scalability — Red-Black Tree

Runnable processes live in a **balanced red-black tree** keyed by `vruntime`. Operations:

- **Pick next** (lowest vruntime): walk to leftmost leaf. O(log n).
- **Insert** (wake-up): O(log n).
- **Remove** (sleep or exit): O(log n).

Logarithmic time matters: datacenter servers have thousands of threads, and a linear scan would waste CPU on scheduling itself.

> [!info] Scheduler overhead matters
> Kanev et al. showed that scheduling alone consumes ~5% of CPU in Google datacenters. Shaving it is real money.

## Sleeping / I/O Handling

**Problem**: if process B sleeps 10 s while A runs, B's vruntime stays stuck 10 s behind. On wake, picking "lowest vruntime" would give B the CPU for 10 s straight — **starving A**.

**Fix**: on wake-up, set B's `vruntime = min(vruntime in tree)`. B gets to run immediately but can't monopolize.

**Cost**: processes that sleep **frequently** for short periods don't accumulate any "owed" CPU, so they get less than their fair share. Trade-off.

## What CFS Doesn't Do

- No adaptive priority learning (unlike [[MLFQ]]).
- Doesn't deeply prefer interactive jobs — if your workload is heavily interactive *and* has heavy background CPU, CFS can feel less snappy than an MLFQ variant.

## Comparison

| | [[MLFQ]] | CFS |
|---|---|---|
| Strategy | Adaptive priority | Fair share via vruntime |
| Good at interactive latency | **Yes** | OK |
| Good at bulk throughput | OK | **Yes** |
| Scales to 10k+ processes | Harder | **Yes** (RB tree) |
| Parameter tuning | [[Ousterhout's Law|Voodoo]] | `nice`-based; less magic |

## Related Notes

- [[Proportional Share Scheduling]], [[Lottery Scheduling]], [[Stride Scheduling]].
- [[MLFQ]] — the other major family.
- [[Scheduler]], [[Scheduling Metrics]].
- [[Ch 9 — Scheduling - Proportional Share]].
