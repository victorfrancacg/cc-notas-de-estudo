---
title: MOC - CPU Virtualization
tags:
  - moc
  - ostep
  - virtualization
  - cpu
aliases:
  - CPU Virtualization
  - OSTEP Part I CPU
type: moc
part: I - Virtualization (CPU)
chapters: 1-10
status: complete
---

# MOC — CPU Virtualization

> [!abstract] The Crux of CPU Virtualization
> **How can the OS provide the illusion of many CPUs when there are only a few (or just one)?** By time-sharing: rapidly switching the CPU between processes, so each one *seems* to run continuously. Everything in Part I is the machinery that makes this illusion cheap, safe, and fair.

Parent: [[MOC - OSTEP]]

## Chapters

1. [[Ch 1 — A Dialogue on the Book]] — why OSTEP exists; the three pieces.
2. [[Ch 2 — Introduction to Operating Systems]] — OS as resource manager; virtualization, concurrency, persistence; design goals; brief history.
3. [[Ch 3 — A Dialogue on Virtualization]] — intuition for the virtualization trick.
4. [[Ch 4 — The Abstraction - The Process]] — what a process is, states, data structures.
5. [[Ch 5 — Interlude - Process API]] — `fork()`, `exec()`, `wait()` and why they're separate.
6. [[Ch 6 — Mechanism - Limited Direct Execution]] — user/kernel mode, traps, timer interrupts, context switch.
7. [[Ch 7 — Scheduling - Introduction]] — FIFO, SJF, STCF, Round Robin; turnaround vs response.
8. [[Ch 8 — Scheduling - MLFQ]] — Multi-Level Feedback Queue: learning workload behavior.
9. [[Ch 9 — Scheduling - Proportional Share]] — lottery, stride, CFS.
10. [[Ch 10 — Multiprocessor Scheduling]] — cache affinity, SQMS, MQMS.
11. *Summary Dialogue on CPU Virtualization — intentionally skipped (Corbató would approve — the real review is just reading the chapters back with this MOC in hand).*

## Core Concepts Introduced in Part I

### The Process
- [[Process]] — the core abstraction.
- [[Address Space]] — what each process sees of memory.
- [[Process Control Block]] — kernel's per-process bookkeeping.
- [[Process States]] — running, ready, blocked.
- [[Context Switch]] — swapping one process for another.

### Process API
- [[fork()]] — create a copy of the current process.
- [[exec()]] — replace the process image with a new program.
- [[wait()]] — parent blocks until a child finishes.
- [[Zombie Process]], [[Orphan Process]] — edge cases of the lifecycle.
- [[Shell]] — user-level program built on `fork`/`exec`/`wait`.

### Mechanism: Limited Direct Execution
- [[Limited Direct Execution]] — direct execution, but with guardrails.
- [[User Mode vs Kernel Mode]] — hardware protection levels.
- [[System Call]] — controlled entry into the kernel.
- [[Trap]], [[Trap Table]] — hardware-assisted kernel entry.
- [[Timer Interrupt]], [[Interrupt]] — how the OS regains control.
- [[Cooperative vs Preemptive Scheduling]].
- [[Kernel]] — the privileged core.

### Policy: Scheduling
- [[Scheduling Metrics]] — turnaround time, response time.
- [[FIFO Scheduling]] — first come, first served.
- [[SJF Scheduling]] — shortest job first.
- [[STCF Scheduling]] — shortest time-to-completion first (preemptive SJF).
- [[Round Robin]] — time-slice rotation.
- [[MLFQ]] — Multi-Level Feedback Queue.
- [[Lottery Scheduling]] — randomness for proportional share.
- [[Stride Scheduling]] — deterministic proportional share.
- [[CFS - Completely Fair Scheduler]] — Linux's default.

### Multiprocessor Scheduling
- [[Cache Affinity]] — keep a process on the same CPU.
- [[Cache Coherence]] — hardware keeping caches in sync.
- [[Single-Queue Multiprocessor Scheduling (SQMS)]].
- [[Multi-Queue Multiprocessor Scheduling (MQMS)]].
- [[Load Balancing (Scheduling)]] — migration between queues.

### Cross-Cutting Design Themes
- [[Mechanism vs Policy]] — separation is a recurring design principle.
- [[Principle of Least Privilege]] — minimum authority needed.
- [[Isolation (OS)]] — keeping bugs from spreading.
- [[Time Sharing]] vs [[Space Sharing]] — two ways to share a resource.
- [[Ousterhout's Law]] — avoid voodoo constants.

## Contrast & Comparison Notes

- [[Scheduler Comparison Table]] — FIFO vs SJF vs STCF vs RR vs MLFQ vs Lottery vs CFS.
- [[Turnaround vs Response Time]] — why you can't optimize both.

## Classic Pitfalls

> [!warning] The [[Convoy Effect]]
> Short jobs stuck behind a long one in [[FIFO Scheduling]]. A non-preemptive queue always risks this.

> [!warning] [[Gaming the Scheduler]]
> Why naïve [[MLFQ]] loses information across jobs — and why cumulative accounting fixes it.

> [!warning] [[Starvation (Scheduling)]]
> The price of always preferring some class of job: the deferred class eventually gets nothing.

> [!warning] Assumptions Leak
> Every scheduler chapter opens with workload assumptions ("jobs are known", "all arrive at time 0", "no I/O"...) and the chapter's progress is mostly about *dropping* those assumptions. Track which assumptions survive.
