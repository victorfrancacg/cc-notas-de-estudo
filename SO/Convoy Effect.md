---
title: Convoy Effect
tags:
  - concept
  - ostep
  - scheduling
  - pitfall
type: concept
introduced_in: Ch 7
---

# Convoy Effect

> [!abstract] Definition
> The **convoy effect** is the pathology where short or lightweight jobs get queued behind a long or heavyweight job in a non-preemptive scheduler, dragging their response and turnaround times up.

## The Name

From [B+79] "The Convoy Phenomenon" (Blasgen et al., 1979), originally about database lock contention. Generalized to any queue where one heavyweight blocks many lightweights.

## Canonical Example

Non-preemptive [[FIFO Scheduling|FIFO]]:

- A arrives at t=0, runs for 100 s.
- B arrives at t=ε, runs for 10 s.
- C arrives at t=2ε, runs for 10 s.

| Job | Completes at | Turnaround |
|---|---:|---:|
| A | 100 | 100 |
| B | 110 | 110 |
| C | 120 | 120 |

Average turnaround = **110 s**. B and C, despite needing only 10 s each, waited 100 s because A was ahead of them in the queue.

## Mental Image

Picture a grocery express lane that isn't actually "10 items or less" — one family with three full carts is ahead of you. You're stuck. That's the convoy effect.

## How Different Policies Handle It

| Policy | Convoy-prone? |
|---|---|
| [[FIFO Scheduling]] | **Yes.** |
| [[SJF Scheduling]] | Yes *if* the long job arrived first (non-preemptive). |
| [[STCF Scheduling]] | **No** — preemption rescues late-arrived short jobs. |
| [[Round Robin]] | No — time slicing gives everyone regular access. |
| [[MLFQ]] | No — learns to demote CPU hogs. |

## Where It Appears Outside Scheduling

The convoy phenomenon shows up anywhere a shared resource is acquired one-at-a-time:

- Database lock queues.
- I/O request queues to a slow disk.
- Printer spoolers.
- Network packet queues (TCP head-of-line blocking).

## Mitigations

- **Preemption** — the single best cure if the resource supports it.
- **Priority / weighting** — let short jobs skip ahead.
- **Multiple queues** — short jobs and long jobs in separate queues (like express lanes).

## Related Notes

- [[FIFO Scheduling]] — the archetypal victim.
- [[STCF Scheduling]], [[Round Robin]] — the preemptive cures.
- [[Scheduling Metrics]] — how convoy effect shows up in averages.
- [[Ch 7 — Scheduling - Introduction]].
