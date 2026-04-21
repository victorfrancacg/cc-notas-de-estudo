---
title: Multiprogramming
tags:
  - concept
  - ostep
  - history
  - virtualization
type: concept
introduced_in: Ch 2
---

# Multiprogramming

> [!abstract] Definition
> **Multiprogramming** is the OS technique of loading **several jobs into memory at once** and switching among them rapidly to keep the CPU busy — especially while one job waits on slow I/O.

## Why It Was a Big Deal

Before multiprogramming (batch era): one job at a time. When that job issued I/O, the CPU sat idle. On a $1000/hour mainframe, idle CPU was real money burning.

Multiprogramming was the fix:

1. Load several jobs into memory.
2. While job A waits for its disk read, switch to job B.
3. Keep the CPU hot.

Massive jump in **utilization**.

## What Multiprogramming Forced the OS to Invent

Multiprogramming is the *parent* of most of what you'll study in OSTEP:

| Problem born from multiprogramming | OS response |
|---|---|
| Programs can read each other's memory | [[Memory Protection]] |
| I/O wait shouldn't pin the CPU | [[Interrupt]]-driven I/O |
| Whose turn to run? | [[Scheduler]], [[Scheduling Metrics]] |
| Programs interfering with each other | [[Isolation (OS)]] |
| Bugs at unpredictable switch points | [[Concurrency]] problems |

> [!info] Historical beat
> Multiprogramming took off in the **minicomputer era** (PDPs at Bell Labs, DEC). UNIX grew up on these machines — and most of what UNIX invented was in service of running many users' programs on one shared box.

## Multiprogramming vs Multitasking vs Time Sharing

Colloquially all used interchangeably, but strictly:

- **Multiprogramming** — many jobs resident, CPU switches *on I/O*.
- **[[Time Sharing]]** — CPU switches *on time slices*, proactively (preemptively). Requires a [[Timer Interrupt]].
- **Multitasking** — modern umbrella term; usually implies time sharing.

All three are about "many things making progress on one CPU." Time sharing is strictly more aggressive than multiprogramming.

## Related Notes

- [[Time Sharing]], [[Context Switch]], [[Scheduler]].
- [[Concurrency]] — the problems multiprogramming exposed to everyone.
- [[Memory Protection]], [[Isolation (OS)]].
- [[Operating System]] history.
