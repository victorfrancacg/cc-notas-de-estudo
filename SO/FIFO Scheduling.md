---
title: FIFO Scheduling
tags:
  - concept
  - ostep
  - scheduling
  - algorithm
aliases:
  - FCFS
  - First Come First Served
  - First In First Out Scheduling
type: concept
introduced_in: Ch 7
---

# FIFO Scheduling

> [!abstract] Definition
> **FIFO** (First In, First Out), a.k.a. **FCFS** (First Come, First Served), runs jobs in the order they arrive. Non-preemptive: once a job starts, it runs to completion.

## Algorithm

```
ready_queue = []

on job arrival: ready_queue.append(job)
on CPU idle: run ready_queue.popleft() to completion
```

Couldn't be simpler.

## When It Works Well

Under the naive workload assumptions (Ch 7 §7.1):

- All jobs run for the same time.
- All arrive simultaneously.
- No preemption needed.

Then FIFO gives decent turnaround, matching any sensible non-preemptive scheduler.

### Example

Three jobs A, B, C of 10s each, all arriving at t=0:

| Job | Completes at |
|---|---:|
| A | 10 |
| B | 20 |
| C | 30 |

Average turnaround = (10+20+30)/3 = **20 s**.

## Where It Fails — The [[Convoy Effect]]

Drop the "same length" assumption. A=100s, B=10s, C=10s, all arrive at t=0:

| Job | Completes at |
|---|---:|
| A | 100 |
| B | 110 |
| C | 120 |

Average turnaround = **110 s** — dominated by A. Short jobs are stuck behind the heavy consumer.

This is the **[[Convoy Effect]]**: short jobs bottlenecked by a long-running predecessor.

## Why You Still Care

- It's the mental baseline. Every other policy is judged against it.
- It's the default for tasks with **equal weight** and **equal size** (batch jobs in early systems).
- It's a cheap ingredient inside other schedulers (e.g., [[MLFQ]] uses FIFO within the lowest queue).

## Pros and Cons

| Pros | Cons |
|---|---|
| Trivial to implement. | Bad under varied job lengths. |
| No preemption → no [[Context Switch]] overhead. | Bad response time. |
| Fair in FIFO-arrival sense (order is respected). | Arrival order isn't actually fair to interactive users. |

## Related Notes

- [[Scheduler]], [[Scheduling Metrics]].
- [[Convoy Effect]] — the main failure mode.
- [[SJF Scheduling]] — the direct improvement.
- [[Round Robin]] — the preemptive generalization.
- [[Ch 7 — Scheduling - Introduction]].
