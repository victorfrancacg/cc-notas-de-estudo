---
title: SJF Scheduling
tags:
  - concept
  - ostep
  - scheduling
  - algorithm
aliases:
  - SJF
  - Shortest Job First
type: concept
introduced_in: Ch 7
---

# SJF Scheduling

> [!abstract] Definition
> **SJF** (Shortest Job First) runs the job with the **shortest known run-time** first, then the next shortest, and so on. Non-preemptive: once a job starts, it runs to completion.

## Algorithm

```
on CPU idle:
    pick the ready job with the smallest known run-time
    run it to completion
```

## Why It Helps

SJF neutralizes the [[Convoy Effect]]. Short jobs go first; their turnaround is small; the long job suffers the same absolute time but doesn't drag everyone else.

### Example

A=100s, B=10s, C=10s, all arrive at t=0. With SJF:

| Job | Completes at |
|---|---:|
| B | 10 |
| C | 20 |
| A | 120 |

Average turnaround = **50 s** (vs 110 s with FIFO — a 2.2× win).

## Provably Optimal… Under Strict Conditions

> [!tip] SJF is optimal for average turnaround IF:
> 1. All jobs arrive at the same time.
> 2. Job run-times are known in advance.
> 3. No preemption needed.
>
> Drop any one and SJF loses optimality.

## Where It Fails — Late Arrivals

Relax assumption 2 of Ch 7 (all arrive at t=0):

- A arrives at t=0, run-time 100s.
- B, C arrive at t=10, run-time 10s each.

SJF has already started A (it was alone). Non-preemptive, so B and C wait until t=100.

| Job | Completes at | Turnaround |
|---|---:|---:|
| A | 100 | 100 |
| B | 110 | 100 |
| C | 120 | 110 |

Average = 103.3 s. The convoy effect is back, because non-preemption can't react to better jobs arriving mid-execution.

The fix: **preemption** — see [[STCF Scheduling]].

## The Oracle Problem

> [!warning] How do you know the run-time?
> You usually don't. A general-purpose OS can't predict how long a user's program will run.
>
> This is the assumption [[MLFQ]] (Ch 8) ultimately drops — by *learning* run-time behavior from past CPU usage.

## Outside Operating Systems

SJF principle generalizes: grocery store express lanes ("10 items or less") apply SJF to shoppers. Most queuing systems informally favor shortest jobs when latency matters.

## Related Notes

- [[FIFO Scheduling]], [[STCF Scheduling]], [[Round Robin]].
- [[Convoy Effect]] — what SJF mitigates.
- [[Scheduling Metrics]].
- [[MLFQ]] — approximates SJF without needing an oracle.
- [[Ch 7 — Scheduling - Introduction]].
