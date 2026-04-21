---
title: Starvation (Scheduling)
tags:
  - concept
  - ostep
  - scheduling
  - pitfall
aliases:
  - Scheduling Starvation
  - Process Starvation
type: concept
introduced_in: Ch 8
---

# Starvation (Scheduling)

> [!abstract] Definition
> **Starvation** occurs when a process, although ready to run, is **never chosen** (or chosen so rarely it makes no meaningful progress) because higher-priority work always exists. It's a fairness failure mode — the process doesn't crash, it just *never runs*.

## How It Arises

Any scheduler that always prefers some jobs over others risks starving the "others" if the preferred class is never empty:

| Scheduler | Who starves? |
|---|---|
| [[STCF Scheduling]] (strict shortest-first) | Long jobs, forever, if short jobs keep arriving. |
| [[MLFQ]] (without priority boost) | Low-priority CPU-bound jobs if interactive jobs keep arriving. |
| Static priority systems | Low-priority jobs whenever any high-priority job exists. |

## Example — MLFQ Without Boost

Interactive jobs keep flooding the top queue. A CPU-bound job sits at the bottom queue forever:

```
Top:    [I1] [I2] [I3] [I4] [I5] ...   always non-empty
Bottom: [CPU]                           never runs
```

With [[Ousterhout's Law|no priority boost]], CPU's CPU share = 0%. Starvation.

### The Fix — Rule 5

> Every *S* time units, boost every job to the top queue.

Even if *S* = 100 ms and the top queue stays busy, CPU is now guaranteed *some* CPU time in every 100-ms window.

## Starvation vs Deadlock

| | Starvation | [[Deadlock]] |
|---|---|---|
| Affected process state | Ready but never picked | Blocked permanently |
| CPU usage | 0% for victim | 0% for all deadlocked parties |
| Resolution | Fair scheduling / priority aging | Detect + break cycle, or prevent |
| Example | MLFQ without boost | Dining Philosophers |

Both are liveness failures — progress fails — but starvation is policy-driven while deadlock is structural.

## Mitigations

- **Aging** — gradually increase waiting job's priority (informal in MLFQ via Rule 5; explicit in some schedulers).
- **Minimum guaranteed share** — proportional-share schedulers (lottery, stride, [[CFS - Completely Fair Scheduler|CFS]]) ensure every job gets *some* CPU by construction.
- **Round-robin at ties** — ensures peers don't starve each other within a priority level.

## Starvation Outside Scheduling

The term shows up in any shared-resource context:

- **Lock starvation** — a thread never gets a lock because others keep grabbing it first.
- **I/O starvation** — a request queue keeps inserting higher-priority requests.
- **Network starvation** — flow-control / QoS failures.

Same shape: a candidate is perpetually ignored while others are served.

## Related Notes

- [[MLFQ]] — Rule 5 (priority boost) exists to cure starvation.
- [[Ousterhout's Law]].
- [[Scheduling Metrics]] — [[Scheduling Metrics|fairness]] is how starvation is measured.
- [[Lottery Scheduling]], [[CFS - Completely Fair Scheduler]] — proportional-share schedulers sidestep starvation by design.
- [[Deadlock]] — the sibling liveness failure.
