---
title: Stride Scheduling
tags:
  - concept
  - ostep
  - scheduling
  - algorithm
type: concept
introduced_in: Ch 9
---

# Stride Scheduling

> [!abstract] Definition
> **Stride scheduling** is a **deterministic** [[Proportional Share Scheduling|proportional-share]] algorithm. Each job has a **stride** inversely proportional to its share, and a **pass** value that advances by the stride every time the job runs. The scheduler picks the job with the lowest pass.

Introduced by Waldspurger (1995) as the deterministic counterpart to [[Lottery Scheduling]].

## Algorithm

For each job:

```
job.stride = BIG_NUMBER / job.tickets    // high tickets → small stride
job.pass   = 0                           // initially
```

Scheduling decision:

```
pick job with minimum pass
run it for one time slice
job.pass += job.stride
```

## Example

`BIG_NUMBER = 10000`.

| Job | Tickets | Stride (10000/tix) |
|---|---:|---:|
| A | 100 | 100 |
| B | 50 | 200 |
| C | 250 | 40 |

Trace:

| Step | pass(A) | pass(B) | pass(C) | Who runs? |
|---:|---:|---:|---:|---|
| 0 | 0 | 0 | 0 | A (tie-break) |
| 1 | 100 | 0 | 0 | B |
| 2 | 100 | 200 | 0 | C |
| 3 | 100 | 200 | 40 | C |
| 4 | 100 | 200 | 80 | C |
| 5 | 100 | 200 | 120 | A |
| 6 | 200 | 200 | 120 | C |
| 7 | 200 | 200 | 160 | C |
| 8 | 200 | 200 | 200 | reset cycle |

After one cycle: C ran 5×, A ran 2×, B ran 1× — exactly 250:100:50 = 5:2:1. **Deterministic proportions**.

## Stride vs Lottery

| | [[Lottery Scheduling|Lottery]] | Stride |
|---|---|---|
| Determinism | Randomized | Deterministic |
| Short-term fairness | Poor | Exact |
| Long-term fairness | Excellent | Exact |
| Per-job state | Just tickets | Tickets + pass |
| Handles new jobs cleanly | **Yes** | **No** |
| Corner-case resistance | No worst case | Can mis-behave if new-job pass is wrong |

## The "New Job" Problem

The fatal issue for stride: what pass value to give a newly-arriving job?

- Set to **0** → the new job monopolizes CPU until its pass catches up.
- Set to **current minimum** → fair from the start, but loses proportionality for an interval.
- Set to **average** / weighted something → heuristic; no perfect answer.

Lottery sidesteps this: just add the new job's tickets to the total. Next draw, it's fair.

This is why **lottery survives despite being less precise** — it has no global state.

## Strengths

- Exact shares over each cycle.
- Simple invariant to reason about (always pick lowest pass).
- Useful where **reproducibility** matters (e.g., simulations, controlled benchmarks).

## Weaknesses

- Doesn't handle arriving/leaving jobs elegantly.
- Same I/O issues as lottery — sleeping jobs either lose share or come back and dominate.

## Related Notes

- [[Proportional Share Scheduling]], [[Lottery Scheduling]], [[CFS - Completely Fair Scheduler]].
- [[Ch 9 — Scheduling - Proportional Share]].
