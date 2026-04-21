---
title: Scheduler
tags:
  - concept
  - ostep
  - scheduling
aliases:
  - CPU Scheduler
  - OS Scheduler
type: concept
introduced_in: Ch 7
---

# Scheduler

> [!abstract] Definition
> The **scheduler** is the part of the OS that decides **which ready process runs next** on a CPU. It is the **policy** layer built atop the **mechanism** of [[Context Switch]].

## What Data It Operates On

- The [[Process List]], filtered to the ready queue (processes in the **Ready** [[Process States|state]]).
- Per-process metadata: accumulated CPU time, priority, last-run time, niceness, etc.
- System-wide context: number of CPUs, current load, time of day (for aging / boosting).

## The Decision Loop (every tick or reschedule point)

```
if someone's time slice expired or a higher-priority job arrived:
    pick the "best" runnable process by policy
    perform a context switch to it
else:
    keep running current process
```

## Policies Covered in OSTEP Part I

| Chapter | Policy | Character |
|---|---|---|
| 7 | [[FIFO Scheduling]], [[SJF Scheduling]], [[STCF Scheduling]], [[Round Robin]] | Baselines and trade-offs |
| 8 | [[MLFQ]] | Learns from history, no oracle needed |
| 9 | [[Lottery Scheduling]], [[Stride Scheduling]], [[CFS - Completely Fair Scheduler|CFS]] | Proportional share |
| 10 | [[Single-Queue Multiprocessor Scheduling (SQMS)]], [[Multi-Queue Multiprocessor Scheduling (MQMS)]] | Multi-CPU |

## What a Good Scheduler Tries to Optimize

Usually some combination of:

- **Turnaround time** — total time from submission to completion.
- **Response time** — time from submission to first CPU.
- **Fairness** — no job permanently starved.
- **Utilization** — keep the CPU busy.
- **Throughput** — jobs completed per unit time.

Most of these **conflict**. Real-world schedulers are compromises.

## Mechanism vs Policy (Again)

Scheduler is *pure policy*. The [[Context Switch]] machinery, [[Timer Interrupt]], and [[Process Control Block]] structure are *pure mechanism*. Changing scheduler (e.g., from O(1) → CFS in Linux 2.6) doesn't touch the mechanism.

See [[Mechanism vs Policy]].

## Long- vs Short-Term Scheduler

Classical OS texts distinguish:

- **Long-term scheduler** — decides which jobs are admitted to the system (relevant on batch systems). Dead in modern desktops.
- **Short-term scheduler** — the one OSTEP means: which ready process runs right now.
- **Medium-term** — swap processes in/out of RAM to disk.

Modern usage of "scheduler" almost always refers to the short-term one.

## Related Notes

- [[Process]], [[Process States]], [[Process List]].
- [[Context Switch]], [[Timer Interrupt]].
- [[Mechanism vs Policy]].
- [[Scheduling Metrics]].
- [[Ch 7 — Scheduling - Introduction]], [[Ch 8 — Scheduling - MLFQ]], [[Ch 9 — Scheduling - Proportional Share]], [[Ch 10 — Multiprocessor Scheduling]].
