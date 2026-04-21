---
title: Scheduling Metrics
tags:
  - concept
  - ostep
  - scheduling
aliases:
  - Turnaround Time
  - Response Time
  - Fairness
  - Throughput
  - Utilization
type: concept
introduced_in: Ch 7
---

# Scheduling Metrics

> [!abstract] Definition
> The quantitative yardsticks used to compare [[Scheduler|schedulers]]. Different metrics rank policies differently — and optimizing one often hurts another.

## The Main Metrics

### Turnaround Time

$$
T_{\text{turnaround}} = T_{\text{completion}} - T_{\text{arrival}}
$$

Total elapsed time from a job entering the system until it finishes. A **performance** metric. Classically the target of batch schedulers.

### Response Time

$$
T_{\text{response}} = T_{\text{firstrun}} - T_{\text{arrival}}
$$

Time from entering the system until the job **first runs** (not completes). An **interactivity** metric. Introduced when time-shared terminals demanded snappiness.

### Fairness

Qualitative. Formally measurable with [**Jain's Fairness Index**](https://en.wikipedia.org/wiki/Fairness_measure#Jain's_fairness_index):

$$
J = \frac{\left(\sum x_i\right)^2}{n \sum x_i^2}
$$

where $x_i$ is the share (e.g., CPU seconds) received by job $i$.

- $J = 1$: perfectly fair (everyone got equal).
- $J = 1/n$: maximally unfair (one job got all).

### Throughput

Jobs completed per unit time. The OS-wide perspective.

### Utilization

Fraction of time the CPU is busy doing useful work. Closely related to [[Multiprogramming]].

## Metrics Conflict

> [!warning] You can't jointly maximize them
> - **Turnaround** loves running shortest-first ([[SJF Scheduling|SJF]]/[[STCF Scheduling|STCF]]). But that **starves** first-arrived long jobs.
> - **Response time** loves spraying the CPU across all jobs every tick ([[Round Robin|RR]]). But that **stretches** turnaround.
> - **Fairness** penalizes both: shortest-first is unfair to long jobs; performance-maximizing is unfair to low-priority ones.

See [[Turnaround vs Response Time]].

## What OSTEP Uses Most

The chapter on scheduling focuses first on **turnaround**, then introduces **response time** when relaxing assumptions. [[MLFQ]] (Ch 8) explicitly targets *both* — the hardest trick.

## Related Notes

- [[Scheduler]], [[FIFO Scheduling]], [[SJF Scheduling]], [[STCF Scheduling]], [[Round Robin]].
- [[Turnaround vs Response Time]] — the core trade-off.
- [[Ch 7 — Scheduling - Introduction]].
