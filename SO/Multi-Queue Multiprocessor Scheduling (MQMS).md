---
title: Multi-Queue Multiprocessor Scheduling (MQMS)
tags:
  - concept
  - ostep
  - scheduling
  - multiprocessor
aliases:
  - MQMS
  - Multi-Queue Scheduling
  - Per-CPU Run Queue
type: concept
introduced_in: Ch 10
---

# Multi-Queue Multiprocessor Scheduling (MQMS)

> [!abstract] Definition
> **MQMS** gives **each CPU its own ready queue**. When a job arrives, it's placed on exactly one queue (random, round-robin, least-loaded…). Each CPU schedules independently from its own queue — no global lock, no cross-CPU contention in the common case.

## Why It Scales

- **No global queue lock** → no contention bottleneck as CPUs multiply.
- **Cache affinity by construction** → each CPU runs its own jobs repeatedly, keeping caches warm.
- **Localized policy** → each queue can even use a different algorithm if desired.

This is the architecture Linux's O(1) scheduler and [[CFS - Completely Fair Scheduler|CFS]] use (per-CPU run queues).

## The Problem — [[Load Balancing (Scheduling)|Load Imbalance]]

> [!warning] The central weakness
> Jobs finish at different rates. Queues drift into imbalance. If not corrected, some CPUs sit idle while others are overloaded.

### Example

Four jobs (A, B, C, D), two CPUs:

```
Initial:    Q0 -> A -> C       Q1 -> B -> D
Schedule:   CPU0: A C A C ...  CPU1: B D B D ...
```

Now C finishes:

```
After C done:  Q0 -> A          Q1 -> B -> D
```

CPU0 has only A; CPU1 still shares time between B and D. Result: A gets 100% of CPU0, while B and D each get 50% of CPU1. **A is getting twice its fair share.**

Later, A finishes too:

```
After A done:  Q0 -> (empty)    Q1 -> B -> D
```

CPU0 idle. CPU1 still busy. Totally wasteful.

## The Fix — Migration

Periodically, the system must **move jobs between queues** to rebalance. Two strategies:

### Explicit Migration

The OS monitors queue lengths and moves jobs from heavy to light queues.

### Work Stealing

Each under-loaded CPU peeks at neighboring queues. If a neighbor has significantly more jobs, it **steals** one.

```
if my_queue is shorter than peer_queue by more than threshold:
    steal a job from peer_queue
```

Trade-off on peek frequency:

- Too often → peek-overhead costs + some contention creeps back in.
- Too rarely → imbalances persist, system under-utilizes.

> [!tip] This is a [[Ousterhout's Law|voodoo constant]]
> Finding the right peek threshold is workload-dependent and resists clean derivation.

## Continuous Migration Example

Even trickier: three jobs, two CPUs.

```
Q0 -> A        Q1 -> B -> D
```

A single move doesn't help: if we move B to Q0, now Q0 has 2 jobs and Q1 has 1, just flipping the imbalance. The only stable answer is **continuous migration** — jobs shuffle back and forth over time:

```
CPU0:  A A A A B A B A B B B ...
CPU1:  B D B D D D D D A D A ...
```

Each CPU spends roughly the same total time on each job.

## Choosing Which Queue on Arrival

When a new job enters, MQMS picks a queue. Common heuristics:

- **Random** — simple, amortizes well over time.
- **Least loaded** — places on the shortest queue.
- **Previous CPU** (for returning blocked jobs) — preserve affinity.

## Summary

| | [[Single-Queue Multiprocessor Scheduling (SQMS)|SQMS]] | MQMS |
|---|---|---|
| Scalability | Poor at high core counts | **Good** |
| Cache affinity | Poor by default | **Good by default** |
| Load balance | Automatic, locked | **Manual / stealing** |
| Complexity | Low | High |

## Related Notes

- [[Single-Queue Multiprocessor Scheduling (SQMS)]] — the contrast.
- [[Load Balancing (Scheduling)]] — the central operational concern.
- [[Cache Affinity]] — MQMS's main advantage.
- [[CFS - Completely Fair Scheduler]] — Linux implementation using this approach.
- [[Ch 10 — Multiprocessor Scheduling]].
