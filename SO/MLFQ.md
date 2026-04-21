---
title: MLFQ
tags:
  - concept
  - ostep
  - scheduling
  - algorithm
aliases:
  - Multi-Level Feedback Queue
  - Multi-level Feedback Queue Scheduler
type: concept
introduced_in: Ch 8
---

# MLFQ — Multi-Level Feedback Queue

> [!abstract] Definition
> **MLFQ** is a [[Scheduler]] that keeps **multiple priority queues**, runs jobs from the highest non-empty queue, and dynamically **adjusts** each job's priority based on its observed behavior — reducing priority for CPU hogs and (periodically) boosting everyone up to prevent starvation.

It achieves strong performance on both [[Scheduling Metrics|turnaround]] and [[Scheduling Metrics|response time]] **without needing to know job lengths in advance** — by *learning* from history.

## The Five Rules

> [!tip] Memorize these five rules
> They are the canonical definition of MLFQ.

### Rule 1 — Priority wins

If `Priority(A) > Priority(B)`, **A runs** (B doesn't).

### Rule 2 — Ties go to Round Robin

If `Priority(A) = Priority(B)`, A and B share the CPU via [[Round Robin]] at that level's time slice.

### Rule 3 — New jobs enter at the top

Every arriving job is placed at the **highest** priority queue. MLFQ is optimistic — "maybe this is a short interactive job" — and only demotes when evidence accumulates.

### Rule 4 — Demote CPU hogs (with cumulative accounting)

If a job **uses up its time allotment** at the current priority level (regardless of how many times it gave up the CPU voluntarily), its priority is **reduced** — it moves down one queue.

> [!warning] The "cumulative" part is crucial
> An earlier draft (Rule 4a/4b) demoted only on full-slice use, and *reset* the budget on voluntary relinquishment. This was gameable: a job could issue a trivial I/O just before its slice expired and stay at top forever. See [[Gaming the Scheduler]].

### Rule 5 — Periodic priority boost

Every *S* time units, **promote every job back to the top queue**. This prevents:

- **[[Starvation (Scheduling)|Starvation]]** — long jobs get occasional turns even when interactive ones dominate.
- **Behavior changes** — a batch job that has become interactive gets a fresh chance.

## Why It Approximates SJF

- Short jobs arrive at top priority (Rule 3), finish before being demoted → **behave like SJF**.
- Long jobs sink to lower queues → don't delay shorter new jobs.
- Fairness emerges from Rule 5.

The scheduler never knows job lengths; it *infers* them from observed CPU usage.

## Why It Approximates Round Robin

- Within a priority level (Rule 2), jobs round-robin.
- Interactive jobs tend to cluster at the top (they give up CPU quickly and get re-promoted), where they round-robin with fine time slices → good response time.

## Typical Queue Configuration

| Queue | Time slice | Role |
|---|---|---|
| Highest | ~10 ms | Interactive jobs |
| Middle | ~20 ms | Mixed |
| Lowest | ~100+ ms | CPU-bound batch |

Shorter slices at top = snappier response. Longer slices at bottom = amortized [[Context Switch]] cost.

## Example Walkthrough

**Scenario 1 — Lone CPU-bound job**

Enters Q2 (top). Runs, uses allotment → demotes to Q1. Runs more, uses allotment → demotes to Q0. Stays there, running RR with any other bottom-feeders. Every *S*, it gets boosted back to Q2 briefly.

**Scenario 2 — CPU-bound A + arriving interactive B**

A is at Q0 (bottom). B arrives at Q2 (top). Rule 1: B runs first. B finishes in one slice. A resumes at Q0. Result: B's turnaround is tiny, A's is barely affected.

**Scenario 3 — Job with I/O**

Every time the job does I/O, it blocks; when it returns, the scheduler checks cumulative usage at this level. If usage hasn't hit the allotment, the job stays at the same priority. So interactive/I/O-bound jobs naturally retain high priority.

## Tuning Challenges

- How many queues?
- What time slice per queue?
- What allotment per level?
- How often to boost (*S*)?

> [!warning] [[Ousterhout's Law|Voodoo constants]]
> These parameters resist principled derivation. Real systems tune them by running workloads and watching metrics.

## Real-World Users

- **BSD UNIX** variants (since late 1980s).
- **Solaris** (Time-Sharing class — table-driven parameters).
- **Windows NT** and its successors.
- **Linux** historically; replaced by [[CFS - Completely Fair Scheduler|CFS]] in 2.6.23 (but MLFQ principles still influence many modern schedulers).

## Related Notes

- [[Scheduler]], [[Scheduling Metrics]].
- [[SJF Scheduling]], [[STCF Scheduling]], [[Round Robin]].
- [[Gaming the Scheduler]], [[Starvation (Scheduling)]], [[Ousterhout's Law]].
- [[CFS - Completely Fair Scheduler]] — the modern Linux alternative.
- [[Ch 8 — Scheduling - MLFQ]].
