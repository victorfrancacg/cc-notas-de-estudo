---
title: Ch 7 — Scheduling - Introduction
tags:
  - ostep
  - chapter
  - part/virtualization
  - scheduling
type: chapter
book: OSTEP
chapter: 7
pages: 73-86
---

# Ch 7 — Scheduling: Introduction

> [!abstract] One-sentence summary
> A tour of fundamental CPU scheduling policies — [[FIFO Scheduling|FIFO]], [[SJF Scheduling|SJF]], [[STCF Scheduling|STCF]], [[Round Robin]] — using a ladder of workload assumptions. Each policy is great for *one* metric ([[Scheduling Metrics|turnaround or response time]]) and lousy for the other.

## Crux of the Problem

> [!example] How to Develop Scheduling Policy?
> What key assumptions? What metrics matter? What basic approaches have been used?

## The Workload Assumptions Ladder

The chapter starts brutally unrealistic and drops assumptions one by one:

| # | Assumption | Relaxed in |
|---|---|---|
| 1 | All jobs run for the same time | §7.4 (SJF) |
| 2 | All jobs arrive at the same time | §7.5 (STCF) |
| 3 | Jobs run to completion once started | §7.5 (STCF preempts) |
| 4 | Jobs only use the CPU (no I/O) | §7.8 |
| 5 | Each job's run time is known ahead of time | §7.9 → [[Ch 8 — Scheduling - MLFQ|MLFQ]] |

> [!tip] A good OS chapter moves assumptions, not just answers
> Track which assumption each new policy drops. That's the chapter's real story.

## Scheduling Metrics

- **[[Scheduling Metrics|Turnaround time]]**: `T_completion - T_arrival`. Performance metric.
- **[[Scheduling Metrics|Response time]]**: `T_firstrun - T_arrival`. Interactivity metric.
- **Fairness** (e.g., Jain's Fairness Index) — often at odds with performance.

## The Four Policies in This Chapter

| Policy | Idea | Pros | Cons |
|---|---|---|---|
| [[FIFO Scheduling|FIFO]] (FCFS) | Run in arrival order | Dirt simple | [[Convoy Effect]] with unequal jobs |
| [[SJF Scheduling|SJF]] | Run shortest (known) job first | Optimal turnaround under §1–§3 | Non-preemptive; needs oracle |
| [[STCF Scheduling|STCF]] (PSJF) | Preemptive SJF — always run job with least remaining time | Optimal turnaround under §1–§3 | Bad response time; still needs oracle |
| [[Round Robin]] | Time-slice among ready jobs | Great response time | Poor turnaround time |

## The Fundamental Trade-Off

> [!warning] Can't have your cake and eat it too
> Optimizing **turnaround** (SJF/STCF) *versus* optimizing **response** (RR) is a real conflict. A scheduler that is fair on a small time scale hurts turnaround; one that lets shortest jobs finish first hurts response for the remaining jobs.

See [[Turnaround vs Response Time]].

## 7.8 Incorporating I/O

When a job does I/O, it **blocks**. Scheduler treats each 10-ms CPU burst as a sub-job:

- Without awareness: one job runs to completion, then the next → poor utilization.
- With overlap: while A does I/O, run B. When A becomes ready again, STCF picks the shorter burst (A).

This is how schedulers naturally treat **I/O-bound jobs as "interactive"** — they get to run quickly, and CPU-bound jobs fill in the gaps.

> [!tip] Overlap enables higher utilization
> Don't let the CPU sit idle while waiting on I/O. Switch to *any* other runnable work.

## 7.9 No More Oracle — Lead-In to Ch 8

A real OS doesn't know job lengths. Next chapter builds [[MLFQ]], a scheduler that **learns** job behavior from the recent past.

## Tip — Amortize the Context-Switch Cost

Context-switch cost includes hidden cache/TLB/branch-predictor flushing. If time slice = 10 ms and switch cost = 1 ms, 10% of CPU is burned on switching. Raise slice to 100 ms → under 1% on switching, but response time suffers.

**Choose time slice as the smallest value that keeps switching overhead negligible.**

## Related Notes

- [[Scheduler]], [[Scheduling Metrics]].
- [[FIFO Scheduling]], [[SJF Scheduling]], [[STCF Scheduling]], [[Round Robin]].
- [[Convoy Effect]], [[Turnaround vs Response Time]].
- [[Ch 8 — Scheduling - MLFQ]] — next step: learning without an oracle.
- [[MOC - CPU Virtualization]].
