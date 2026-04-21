---
title: Ch 10 — Multiprocessor Scheduling
tags:
  - ostep
  - chapter
  - part/virtualization
  - scheduling
  - multiprocessor
type: chapter
book: OSTEP
chapter: 10
pages: 113-124
---

# Ch 10 — Multiprocessor Scheduling (Advanced)

> [!abstract] One-sentence summary
> With multiple CPUs come three new headaches: **[[Cache Coherence]]** (mostly the hardware's problem), **synchronization** (locks become contended at scale), and **[[Cache Affinity]]** (keep processes on the same CPU to exploit warm caches). The scheduler picks between **[[Single-Queue Multiprocessor Scheduling (SQMS)|SQMS]]** (simple, doesn't scale) and **[[Multi-Queue Multiprocessor Scheduling (MQMS)|MQMS]]** (scales, needs [[Load Balancing (Scheduling)|load balancing]]).

## Why This Chapter Is "Advanced"

OSTEP labels it advanced because it depends on concurrency concepts from [[Concurrency|Part II]]. You can read it now for scheduling context, or revisit after threads and locks.

## Crux of the Problem

> [!example] How to Schedule Jobs on Multiple CPUs?
> How should the OS schedule on multi-CPU hardware? What new problems arise? Do single-CPU techniques still work?

## 10.1 Background — Multiprocessor Architecture

Key difference from single-CPU: **caches**.

- Each CPU has its own cache (fast, small).
- Main memory is shared, larger, slower.
- A piece of data can be in multiple CPUs' caches at once.
- If one CPU writes that data, others must see the update.

That last requirement is **[[Cache Coherence]]**. Hardware (typically via **bus snooping**) takes care of it — caches watch memory traffic and invalidate/update their copies on writes.

## 10.2 Don't Forget Synchronization

Cache coherence preserves memory *visibility* but doesn't prevent **races**. Example:

```c
Node_t *List_Pop() {
    Node_t *tmp = head;
    int value = head->value;
    head = head->next;
    free(tmp);
    return value;
}
```

Two threads entering concurrently can both read the same `head`, both free it (double-free), both return the same value. Fix: a lock (`mutex`) around the critical section.

> [!warning] Locks become bottlenecks as CPU count grows
> As more CPUs contend for the same lock, more time is spent waiting instead of computing. Scalable multi-CPU systems avoid globally-contended locks.

## 10.3 [[Cache Affinity]]

When a process runs on CPU_i, its working set gradually populates CPU_i's caches and TLB. If the scheduler keeps it on CPU_i next time, it runs faster (warm cache). Migrating it to CPU_j forces re-warming on CPU_j — slower.

**Cache affinity** is the design principle: prefer keeping a process on the same CPU when possible.

## 10.4 [[Single-Queue Multiprocessor Scheduling (SQMS)|Single-Queue Scheduling (SQMS)]]

- One global queue; every CPU pulls the next job from it.
- **Pros**: simple; reuses single-CPU scheduler logic directly.
- **Cons**:
  - **Scalability** — the queue's lock is contended.
  - **Cache affinity** — jobs bounce between CPUs, caches stay cold.
- Affinity workaround: stickiness heuristics — try to return a process to its prior CPU. Not free.

## 10.5 [[Multi-Queue Multiprocessor Scheduling (MQMS)|Multi-Queue Scheduling (MQMS)]]

- One queue per CPU (or per cluster).
- New jobs placed on some queue (random, least-loaded, …).
- Each CPU schedules **its own** queue independently — no global lock contention.
- **Pros**: scales, preserves cache affinity by construction.
- **Cons**: **[[Load Balancing (Scheduling)|load imbalance]]** — some queues empty while others are full.

### Handling Imbalance: Migration

- Simple case: CPU A idle, CPU B has two jobs → move one to A.
- Harder case: four jobs, two CPUs, one job finishes → remaining three jobs split 2-1 → one CPU does 2×, the other 1×. The fair fix requires **continuous migration**, shuffling jobs back and forth to even out share.

### Work Stealing

A lightly-loaded CPU's queue *peeks* at a neighbor queue; if the neighbor has more, it steals a job. Key tuning: **how often to peek?**

- Too often → overhead, lock contention creeps back in.
- Too rarely → imbalance persists, system under-utilized.

Sweet spot is workload-dependent ([[Ousterhout's Law|voodoo constant]] territory).

## 10.6 Linux Multiprocessor Schedulers

No single winner has emerged:

| Scheduler | Queue structure | Style |
|---|---|---|
| **O(1)** (older) | Multi-queue | Priority-based, similar to [[MLFQ]] |
| **[[CFS - Completely Fair Scheduler|CFS]]** (current) | Multi-queue | Proportional-share, deterministic |
| **BFS** (out-of-tree) | Single-queue | Proportional-share (EEVDF) |

Both queue structures can work — shows how orthogonal queue choice is to policy.

## 10.7 Summary

Building a general-purpose multiprocessor scheduler is a **daunting task**:

- Small code changes → large behavioral differences.
- Trade-off space is multi-dimensional (throughput, fairness, affinity, latency).
- No optimum exists; only reasonable compromises.

## Related Notes

- [[Cache Affinity]], [[Cache Coherence]].
- [[Single-Queue Multiprocessor Scheduling (SQMS)]], [[Multi-Queue Multiprocessor Scheduling (MQMS)]].
- [[Load Balancing (Scheduling)]] — migration & work stealing.
- [[Scheduler]], [[CFS - Completely Fair Scheduler]], [[MLFQ]].
- [[MOC - CPU Virtualization]].
