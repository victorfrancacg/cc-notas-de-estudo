---
title: Single-Queue Multiprocessor Scheduling (SQMS)
tags:
  - concept
  - ostep
  - scheduling
  - multiprocessor
aliases:
  - SQMS
  - Single-Queue Scheduling
type: concept
introduced_in: Ch 10
---

# Single-Queue Multiprocessor Scheduling (SQMS)

> [!abstract] Definition
> **SQMS** is the simplest multiprocessor scheduling architecture: **one shared queue** of ready jobs that every CPU pulls from. Each CPU, when it needs work, locks the queue, picks the next job by whatever policy you like, and runs it.

## Pros

- **Simple to implement** — reuses a single-CPU scheduler directly.
- **Automatic load balancing** — idle CPUs just grab work from the queue.
- **Easy policy changes** — plug in any single-CPU policy (MLFQ, CFS, etc.) and you have a multiprocessor version.

## Cons

### Scalability

> [!warning] The queue lock is a bottleneck
> Every CPU has to lock the shared queue to pick a job. As CPU count grows, lock contention grows. On a 64-core machine, the CPU might spend a large fraction of its time spinning on that one lock — defeating the point of having many CPUs.

### [[Cache Affinity]]

In a naïve SQMS, a job runs on whichever CPU happens to grab it next. Across repeated scheduling rounds, a job bounces between CPUs:

```
Round 1:   CPU0 gets A, CPU1 gets B, CPU2 gets C
Round 2:   CPU0 gets D, CPU1 gets A, CPU2 gets E
Round 3:   CPU0 gets B, CPU1 gets E, CPU2 gets A
...
```

Every job sees a **cold cache** on every scheduling round. Performance suffers.

### Affinity Heuristics

A real SQMS adds stickiness: try to reassign each job to the CPU it last ran on, and only move *some* jobs to balance. This complicates the simple design and isn't perfect:

- Some jobs get to stay home → good affinity.
- Others get shuffled → uneven penalties.
- Fairness becomes trickier.

## Summary — Where SQMS Fits

| | Good for | Bad for |
|---|---|---|
| Small-core systems (2–4 CPUs) | Yes | — |
| Datacenter servers | — | Yes |
| Research / teaching context | Yes (easy to explain) | — |
| Production kernels | **No** at modern scale | |

The contrast is [[Multi-Queue Multiprocessor Scheduling (MQMS)]] — one queue per CPU, scales better but has its own headaches.

## Related Notes

- [[Multi-Queue Multiprocessor Scheduling (MQMS)]] — the alternative.
- [[Cache Affinity]] — the main performance concern SQMS struggles with.
- [[Load Balancing (Scheduling)]] — handled automatically (but not efficiently).
- [[Ch 10 — Multiprocessor Scheduling]].
