---
title: STCF Scheduling
tags:
  - concept
  - ostep
  - scheduling
  - algorithm
aliases:
  - STCF
  - Shortest Time-to-Completion First
  - Preemptive SJF
  - PSJF
type: concept
introduced_in: Ch 7
---

# STCF Scheduling

> [!abstract] Definition
> **STCF** (Shortest Time-to-Completion First), also known as **PSJF** (Preemptive SJF), always runs the job with the **least remaining work**. On any new job's arrival, the scheduler re-evaluates and may **preempt** the currently running job if the newcomer has less work to finish.

## Algorithm

```
on ready_queue change (arrival or completion):
    pick job with smallest remaining run-time
    if different from currently running job:
        preempt and context-switch to it
```

## Why It Beats SJF

SJF is non-preemptive: once a long job starts, shorter late-arrivers wait.

STCF preempts: if a 10-ms job arrives while a 100-s job is 2 seconds in, STCF *pauses* the long job and runs the short one, then resumes.

### Example

A arrives at t=0, run-time 100s. B, C arrive at t=10, run-time 10s each.

- t=0..10: A runs (only one in system).
- t=10: B and C arrive. STCF preempts A, runs B.
- t=10..20: B runs to completion.
- t=20..30: C runs to completion.
- t=30..120: A resumes and finishes.

| Job | Completes at | Turnaround |
|---|---:|---:|
| B | 20 | 10 |
| C | 30 | 20 |
| A | 120 | 120 |

Average = **50 s** (vs 103.3 s with SJF on the same workload).

## Provably Optimal (Again, With Caveats)

> [!tip] STCF is optimal for average turnaround when:
> - Job run-times are known.
> - Preemption is free (no context-switch overhead).

Real systems violate both, but STCF remains a strong benchmark.

## The Response-Time Disaster

Optimize turnaround, wreck response time. If 3 jobs arrive at t=0:

- Shortest starts at 0 (response time = 0).
- Middle starts after the shortest (response time = shortest's run-time).
- Longest starts last (response time = sum of shorter two).

Response time = *cumulative* waits. Interactive users sitting at a terminal see this as "the system is frozen" until their turn.

See [[Round Robin]] for the response-time answer and [[Turnaround vs Response Time]] for the trade-off.

## The Oracle Problem (Same As SJF)

STCF inherits SJF's unrealistic requirement: **knowledge of remaining run-time**. [[MLFQ]] is how real schedulers approximate STCF without an oracle — by inferring short/long based on recent behavior.

## Related Notes

- [[SJF Scheduling]] — non-preemptive ancestor.
- [[FIFO Scheduling]], [[Round Robin]].
- [[Scheduling Metrics]].
- [[Turnaround vs Response Time]].
- [[MLFQ]] — adaptive, oracle-free approximation.
- [[Ch 7 — Scheduling - Introduction]].
