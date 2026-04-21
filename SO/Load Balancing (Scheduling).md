---
title: Load Balancing (Scheduling)
tags:
  - concept
  - ostep
  - scheduling
  - multiprocessor
aliases:
  - Migration (Scheduling)
  - Work Stealing
  - Scheduler Load Balance
type: concept
introduced_in: Ch 10
---

# Load Balancing (Scheduling)

> [!abstract] Definition
> **Load balancing** in a multiprocessor OS is the ongoing effort to keep work evenly distributed across CPU run queues, usually by **migrating** jobs from overloaded CPUs to idle or lightly loaded ones.

Most relevant to [[Multi-Queue Multiprocessor Scheduling (MQMS)|MQMS]] — in [[Single-Queue Multiprocessor Scheduling (SQMS)|SQMS]], the single queue auto-balances (at the cost of scalability).

## Why It's Needed

Without load balancing in MQMS:

- Some CPUs become idle while others run at 100%.
- Fairness breaks — jobs on a lightly loaded queue effectively get more CPU than jobs on a heavy one.
- Throughput suffers — idle CPUs are wasted hardware.

## Two Core Approaches

### Push Migration (Explicit)

The OS periodically checks queue lengths. If imbalance exceeds a threshold, it *pushes* jobs from heavy queues to light ones.

```
on periodic balance check:
    find most-loaded Q_src
    find least-loaded Q_dst
    if Q_src.size - Q_dst.size > threshold:
        move some jobs from Q_src to Q_dst
```

### Pull Migration / Work Stealing

Idle or under-loaded CPUs *pull* jobs themselves:

```
on my CPU going idle (or running low):
    pick a random target CPU's queue
    peek its length
    if target has more work than me:
        steal a job
```

Popularized by the Cilk-5 runtime (Frigo et al., 1998). Widely used in language runtimes (Go, Rust, Java ForkJoinPool) and kernels.

## The Threshold Problem

> [!warning] [[Ousterhout's Law|Voodoo constant]] ahead
> How often should CPUs peek at each other? How large must the imbalance be before migrating?
>
> - Too aggressive → overhead, cache cooling from forced migrations, contention if peek involves locking.
> - Too passive → imbalance persists, wasted CPU.
>
> Every OS ships with tuned defaults and then fiddles with them for years.

## Continuous Migration

Even with perfect migration policy, an odd number of jobs on an even number of CPUs causes *continuous* shuffling:

```
3 jobs (A, B, D) on 2 CPUs

Step 1:  CPU0: A        CPU1: B, D   (unbalanced)
Migrate D:
Step 2:  CPU0: A, D     CPU1: B       (now CPU1 under)
Migrate D back or A:
Step 3:  CPU0: A        CPU1: B, D   (or similar)
...
```

Jobs shuffle back and forth forever. Each shuffle costs some [[Cache Affinity|cache-warmth]] — a real performance tax.

## Trade-Off with [[Cache Affinity]]

The two goals fight:

- **Balance** → migrate aggressively to distribute load.
- **Affinity** → keep jobs on home CPUs to keep caches warm.

Real schedulers compromise by migrating only when imbalance significantly outweighs the affinity cost.

## What Linux Does

Linux's CFS has a **per-CPU run queue**, a periodic `load_balance` function, and sophisticated heuristics for deciding when to migrate (NUMA awareness, energy, cache hierarchies). There's no single answer — Linux's balance policy has evolved across kernel versions.

## Related Notes

- [[Multi-Queue Multiprocessor Scheduling (MQMS)]] — where load balancing is crucial.
- [[Single-Queue Multiprocessor Scheduling (SQMS)]] — auto-balanced but doesn't scale.
- [[Cache Affinity]] — the main counter-pressure.
- [[Ousterhout's Law]] — on tuning thresholds.
- [[Ch 10 — Multiprocessor Scheduling]].
