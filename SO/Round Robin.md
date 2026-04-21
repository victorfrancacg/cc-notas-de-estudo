---
title: Round Robin
tags:
  - concept
  - ostep
  - scheduling
  - algorithm
aliases:
  - RR
  - Round-Robin
  - Time Slicing
type: concept
introduced_in: Ch 7
---

# Round Robin

> [!abstract] Definition
> **Round Robin (RR)** runs each ready job for a fixed **time slice** (a.k.a. **scheduling quantum**), then moves it to the back of the queue and runs the next. This repeats until all jobs finish. RR prioritizes **response time** over turnaround time.

## Algorithm

```
ready_queue = []      # FIFO queue of ready processes

on timer tick:
    current job's slice consumed += 1 tick
    if current job's slice is up:
        move current to back of ready_queue
        run ready_queue.popleft()
```

Time slice = integer multiple of the [[Timer Interrupt]] period (e.g., 10 ms if timer is 10 ms).

## Why It's Good for Response Time

Because every job reaches the CPU within (N-1) × slice time of arrival (for N ready jobs). No waiting for long jobs to finish.

### Example

Three jobs A, B, C of 5s each, all arriving at t=0, slice=1s:

```
ABCABCABCABCABC (repeating)
```

Response times: A=0, B=1, C=2. Average **1 s** (vs 5 s with SJF / 15 s sum-of-run-times for worst case).

## Why It's Bad for Turnaround

The very act of interleaving *stretches* every job's completion time. In the above example:

| Job | Completes at |
|---|---:|
| A | 13 |
| B | 14 |
| C | 15 |

Average turnaround = **14 s** (vs 10 s with SJF — every job waits for all others to get a slice). Nearly pessimal.

## The Time-Slice Trade-Off

| Slice length | Response time | Context-switch overhead |
|---|---|---|
| **Very short** (1 ms) | Excellent | Dominates — many switches, cache/TLB pollution |
| **Medium** (10 ms) | Good | Amortized (~1%) |
| **Very long** (1 s) | Starts approximating FIFO | Negligible |

> [!tip] Amortize the switch cost
> Choose the smallest slice that keeps switching overhead negligible. Linux has historically tuned this to the 1–10 ms range.

## Fairness vs Turnaround

RR's even time-sharing is its *fairness*. But fairness at small time scales is **turnaround-hostile**: short jobs never get to "finish quickly" — they always wait for the round.

> [!warning] Any fair policy is bad for turnaround
> This is a generic observation. If you're spraying the CPU evenly, short jobs can't race ahead.

See [[Turnaround vs Response Time]].

## RR + I/O

When a job blocks on I/O, it leaves the ready queue. It returns to the back when I/O completes. This naturally favors interactive I/O-bound jobs — they keep relinquishing their slice early and return for a fresh one.

See also [[MLFQ]], which formalizes this preference.

## Hidden Context-Switch Cost

Beyond saving/restoring registers:

- Cache cooled: next job may fault many cache misses.
- TLB cooled: same for page table translations.
- Branch predictors cooled.

These indirect costs can dwarf the direct register-save/restore cost (the original MB91 paper quantified this).

## Related Notes

- [[Context Switch]], [[Timer Interrupt]].
- [[FIFO Scheduling]], [[SJF Scheduling]], [[STCF Scheduling]].
- [[Turnaround vs Response Time]].
- [[MLFQ]] — builds on RR within each priority queue.
- [[Ch 7 — Scheduling - Introduction]].
