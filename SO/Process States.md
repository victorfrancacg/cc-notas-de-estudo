---
title: Process States
tags:
  - concept
  - ostep
  - process
  - scheduling
aliases:
  - Process State
  - Process Lifecycle
type: concept
introduced_in: Ch 4
---

# Process States

> [!abstract] Definition
> At any instant, a [[Process]] is in exactly one **state**. The OS tracks this in the [[Process Control Block]] and uses it to decide what to do next (schedule? ignore? wake up?).

## The Three Essential States

| State | Meaning |
|---|---|
| **Running** | The process is currently executing on a CPU. |
| **Ready** | The process is ready to run but the [[Scheduler]] chose someone else. |
| **Blocked** | The process is waiting on some event (I/O completion, signal, child exit). It's not eligible to be scheduled until the event arrives. |

## State Transition Diagram

```
                    Scheduled
        +------------------------->+
   [Ready]                          [Running]
        +<-------------------------+
                    Descheduled

   [Running] ---- I/O: initiate ----> [Blocked]
   [Blocked] ---- I/O: done ---------> [Ready]
```

### The Six Transitions (verbs matter)

| From | To | Cause | Name |
|---|---|---|---|
| Ready | Running | Scheduler picks the process | **scheduled** |
| Running | Ready | Timer / preemption | **descheduled** |
| Running | Blocked | Process issues I/O, waits on event | *block* |
| Blocked | Ready | Event completes (e.g., disk finished read) | *unblock* / *wake up* |

> [!tip] Blocked ≠ Ready
> A blocked process is *not* a candidate to run. A ready process is. Confusing them makes scheduling reasoning impossible.

## Why Blocked Exists

Without a **Blocked** state, a process waiting for I/O would sit in Ready and chew CPU every time it's scheduled, just to discover "still waiting". The Blocked state lets the scheduler *skip* it entirely until the event arrives.

This is the core insight behind [[Multiprogramming]]: overlap CPU with I/O by moving waiting processes out of the ready set.

## Extended States (in real OSes)

- **Initial / Embryo** — being created, not yet runnable.
- **[[Zombie Process|Zombie]] / Final** — exited, but parent hasn't `wait()`ed yet. Kept around so the parent can read the exit status.
- **Swapped-out** — suspended, possibly with pages swapped to disk.

## Example Traces

### CPU-only (no I/O)

| Time | P0 | P1 |
|---:|---|---|
| 1 | Running | Ready |
| 2 | Running | Ready |
| 3 | Running | Ready |
| 4 | Running (done) | Ready |
| 5 | — | Running |
| 6 | — | Running |
| 7 | — | Running |
| 8 | — | Running (done) |

P0 hogs the CPU until done; P1 never even starts. OS is happy (100% utilization) but P1 is unhappy.

### With I/O

| Time | P0 | P1 | Notes |
|---:|---|---|---|
| 1 | Running | Ready | |
| 2 | Running | Ready | |
| 3 | Running | Ready | P0 initiates I/O |
| 4 | Blocked | Running | P0 is blocked, P1 runs |
| 5 | Blocked | Running | |
| 6 | Blocked | Running | |
| 7 | Ready | Running | I/O done |
| 8 | Ready | Running (done) | |
| 9 | Running | — | |
| 10 | Running (done) | — | |

Now CPU stays busy *and* P1 got to run — a glimpse of [[Multiprogramming]] in action.

## Related Notes

- [[Process]], [[Process Control Block]], [[Process List]].
- [[Scheduler]] — reads state to make decisions.
- [[Context Switch]] — what happens on the Running↔Ready transitions.
- [[Zombie Process]] — a special terminal state.
