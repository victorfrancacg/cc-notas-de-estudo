---
title: Turnaround vs Response Time
tags:
  - concept
  - ostep
  - scheduling
  - trade-off
type: concept
introduced_in: Ch 7
---

# Turnaround vs Response Time

> [!abstract] The core trade-off
> You can optimize **turnaround** (how fast jobs finish) *or* **response time** (how fast jobs start), but **not both** with one simple policy. Every classic scheduler excels at one and loses at the other. Later schedulers like [[MLFQ]] and [[CFS - Completely Fair Scheduler|CFS]] try to be decent at both.

## The Two Metrics

$$
\begin{aligned}
T_{\text{turnaround}} &= T_{\text{completion}} - T_{\text{arrival}} \\[4pt]
T_{\text{response}} &= T_{\text{firstrun}} - T_{\text{arrival}}
\end{aligned}
$$

See [[Scheduling Metrics]] for full definitions.

## Why They Conflict — The Intuition

- **Optimize turnaround**: finish jobs fast. Best strategy: run the **shortest remaining job first** ([[STCF Scheduling|STCF]]). But if three jobs arrive at t=0, the *third* waits for both others to finish → its response time is bad.
- **Optimize response**: give every job CPU as soon as possible. Best strategy: slice time among all ready jobs ([[Round Robin|RR]]). But now *every* job runs longer than necessary → turnaround is stretched.

## A Concrete Example

Three jobs A, B, C of 5 s each, all at t=0.

### STCF (or SJF) — optimal for turnaround

```
AAAAA BBBBB CCCCC
0    5    10    15
```

| Metric | A | B | C | Avg |
|---|---:|---:|---:|---:|
| Response | 0 | 5 | 10 | **5** |
| Turnaround | 5 | 10 | 15 | **10** |

### Round Robin (slice = 1 s) — optimal for response

```
ABCABCABCABCABC
```

| Metric | A | B | C | Avg |
|---|---:|---:|---:|---:|
| Response | 0 | 1 | 2 | **1** |
| Turnaround | 13 | 14 | 15 | **14** |

Response time improved 5×. Turnaround got 40% worse. Exactly the trade.

## Fair Policies Are Bad for Turnaround

> [!tip] A generalization worth memorizing
> *Any* policy that divides the CPU evenly on a small time scale will have poor turnaround. Fairness (to all jobs, at all times) is inherently hostile to minimizing completion time for the shortest job.

## Schedulers That Try to Bridge the Gap

| Scheduler | Strategy |
|---|---|
| [[MLFQ]] | Learn which jobs are interactive (short, I/O-bound) and prioritize them; let long jobs run when interactive ones are idle. |
| [[CFS - Completely Fair Scheduler]] | Virtual-time fairness with I/O-sensitive hooks; keeps both metrics reasonable. |
| [[Lottery Scheduling]] | Randomness distributes CPU, trading tiny fairness violations for simplicity. |

## The Deeper Lesson

OSTEP returns to this constantly: **every non-trivial OS design is a trade-off between competing [[OS Design Goals|goals]]**. Name the goals, choose the trade, measure the outcome.

## Related Notes

- [[Scheduling Metrics]].
- [[FIFO Scheduling]], [[SJF Scheduling]], [[STCF Scheduling]], [[Round Robin]], [[MLFQ]].
- [[OS Design Goals]].
