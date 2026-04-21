---
title: Scheduler Comparison Table
tags:
  - concept
  - ostep
  - scheduling
  - contrast
  - reference
type: reference
introduced_in: Ch 7-10
---

# Scheduler Comparison Table

> [!abstract] Purpose
> A side-by-side summary of every CPU scheduling policy covered in OSTEP Part I. Use this to recall trade-offs quickly — especially before exams.

## The Schedulers

| Scheduler | Family | Preemptive? | Optimizes | Needs oracle? | Starvation-free? | Scales multi-CPU? |
|---|---|:-:|---|:-:|:-:|:-:|
| [[FIFO Scheduling\|FIFO / FCFS]] | Baseline | No | None (trivial) | No | Yes | N/A |
| [[SJF Scheduling\|SJF]] | Turnaround | No | Turnaround | **Yes** | No (long jobs) | N/A |
| [[STCF Scheduling\|STCF / PSJF]] | Turnaround | Yes | Turnaround | **Yes** | No | N/A |
| [[Round Robin\|RR]] | Response | Yes | Response | No | Yes | OK |
| [[MLFQ]] | Adaptive | Yes | Both | **No** (learns) | Yes (Rule 5) | OK |
| [[Lottery Scheduling\|Lottery]] | Prop-share | Yes | Share ratios | No | Probabilistic | OK |
| [[Stride Scheduling\|Stride]] | Prop-share | Yes | Share ratios | No | Yes | Harder |
| [[CFS - Completely Fair Scheduler\|CFS]] | Prop-share | Yes | Share ratios | No | Yes | **Very well** |

## Key Metric Rankings

### Turnaround time (shorter is better)

1. [[STCF Scheduling|STCF]] — provably optimal if run-times known.
2. [[SJF Scheduling|SJF]] — next-best; loses when jobs arrive late.
3. [[MLFQ]] — approximates STCF via learning.
4. [[CFS - Completely Fair Scheduler|CFS]] / [[Lottery Scheduling|Lottery]] / [[Stride Scheduling|Stride]] — decent; not their goal.
5. [[FIFO Scheduling|FIFO]] — bad (convoy effect).
6. [[Round Robin|RR]] — worst if jobs differ in length.

### Response time (shorter is better)

1. [[Round Robin|RR]] — excellent with short slice.
2. [[MLFQ]] — good (interactive jobs stay at top).
3. [[CFS - Completely Fair Scheduler|CFS]] — good via small slices.
4. [[Lottery Scheduling|Lottery]] / [[Stride Scheduling|Stride]] — acceptable.
5. [[STCF Scheduling|STCF]] / [[SJF Scheduling|SJF]] — poor (later arrivals wait).
6. [[FIFO Scheduling|FIFO]] — terrible.

### Fairness (proportional to share)

1. [[Stride Scheduling|Stride]] — exact proportions.
2. [[CFS - Completely Fair Scheduler|CFS]] — near-perfect, efficient.
3. [[Lottery Scheduling|Lottery]] — probabilistic, converges.
4. [[MLFQ]] — fair in the "no starvation" sense; not share-based.
5. [[Round Robin|RR]] — fair if jobs have equal weight.
6. [[FIFO Scheduling|FIFO]] / [[SJF Scheduling|SJF]] / [[STCF Scheduling|STCF]] — not fairness-oriented.

## Which One To Remember For What

| You want… | Pick |
|---|---|
| "What's the classic textbook scheduler?" | [[MLFQ]] |
| "What does Linux actually use?" | [[CFS - Completely Fair Scheduler|CFS]] |
| "What's provably optimal for turnaround?" | [[STCF Scheduling|STCF]] |
| "What's simplest to implement?" | [[FIFO Scheduling|FIFO]] |
| "What's used for cloud VM CPU guarantees?" | [[Lottery Scheduling|Lottery]], [[Stride Scheduling|Stride]], [[CFS - Completely Fair Scheduler|CFS]] |

## Preemption as a Dividing Line

```
Non-preemptive:        Preemptive:
  FIFO                    STCF, RR, MLFQ,
  SJF                     Lottery, Stride, CFS
```

Non-preemptive schedulers were fine for batch systems; modern interactive systems require preemption enabled by a [[Timer Interrupt]].

## Multiprocessor Layering

Any of the above policies can run atop either:

- [[Single-Queue Multiprocessor Scheduling (SQMS)]] — simple, contended.
- [[Multi-Queue Multiprocessor Scheduling (MQMS)]] — scalable, needs [[Load Balancing (Scheduling)|load balancing]].

Linux's O(1), CFS, and BFS all pick different combinations.

## Related Notes

- [[Scheduler]], [[Scheduling Metrics]], [[Turnaround vs Response Time]].
- Individual scheduler notes linked above.
- [[Ch 7 — Scheduling - Introduction]] through [[Ch 10 — Multiprocessor Scheduling]].
- [[MOC - CPU Virtualization]].
