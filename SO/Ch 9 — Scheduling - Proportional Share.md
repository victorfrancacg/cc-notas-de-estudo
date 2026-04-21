---
title: Ch 9 — Scheduling - Proportional Share
tags:
  - ostep
  - chapter
  - part/virtualization
  - scheduling
type: chapter
book: OSTEP
chapter: 9
pages: 99-112
---

# Ch 9 — Scheduling: Proportional Share

> [!abstract] One-sentence summary
> A family of schedulers that, instead of optimizing turnaround or response time, **guarantee each job a specified fraction of CPU**. Three approaches: **[[Lottery Scheduling]]** (randomized), **[[Stride Scheduling]]** (deterministic), **[[CFS - Completely Fair Scheduler|CFS]]** (Linux default — efficient RB-tree-based fair sharing).

## Crux of the Problem

> [!example] How to Share the CPU Proportionally?
> Design a scheduler to share CPU **in a proportional manner**. What mechanisms enable this? How effective are they?

## The Big Idea — [[Proportional Share Scheduling]]

Instead of optimizing turnaround or response, the objective is: *every job gets the fraction of CPU it was promised*. If A has 3× B's share, A should get roughly 3× more CPU over any meaningful window.

See [[Proportional Share Scheduling]] for the family overview.

## Three Implementations

### [[Lottery Scheduling]] (Waldspurger & Weihl, 1994)

- Each job holds **tickets** representing its share.
- Every time slice, the scheduler picks a random ticket.
- Whoever holds that ticket runs next.
- Statistically: over long runs, jobs get CPU in proportion to their tickets.

**Advantages of randomness:**
- No worst-case pathological workloads (cf. [[LRU]] vs worst-case cyclic workloads).
- Minimal per-process state (just a ticket count).
- Fast — just a random number and a walk of the list.

**Mechanisms on top:**
- **Ticket currency** — users have local tickets; OS converts to a global scale, so a user can't unilaterally inflate.
- **Ticket transfer** — hand tickets to another process temporarily (client → server RPC).
- **Ticket inflation** — in trusted environments, a process can raise/lower its own ticket count.

### [[Stride Scheduling]] (Waldspurger, 1995)

- Each job has a **stride** = `BIG_NUMBER / tickets` (inversely proportional to tickets).
- Each job tracks a **pass** value.
- Scheduler picks the job with the lowest pass, runs it, adds its stride to its pass.
- Result: exact proportions, deterministically.

**But:** has **global state** — when a new job arrives, what pass value to assign? (Setting it to 0 would let it monopolize until it catches up.) This is why lottery survives despite being less precise.

### [[CFS - Completely Fair Scheduler|CFS]] (Linux, 2007)

- Default Linux scheduler since 2.6.23.
- Core concept: **virtual runtime** (`vruntime`) — each process's vruntime accumulates as it runs.
- Always run the process with the **lowest vruntime**.
- Scales to thousands of processes via a **red-black tree** keyed by vruntime (O(log n) ops).

**Key parameters:**
- `sched_latency` (~48 ms) — target window in which every process should get its share.
- `min_granularity` (~6 ms) — floor on a per-process time slice (avoids thrashing with too many processes).

**Priorities via `nice`:**
- `nice` ranges −20 to +19 (negative = higher priority).
- Mapped to a **weight** table; higher weight = vruntime advances slower = chosen more often.

**Sleep / wake-up problem** — if process B sleeps 10 s, its vruntime lags 10 s behind A's. If we just picked "lowest vruntime", B would hog the CPU for 10 s after waking to "catch up" and **starve A**. CFS fixes this by setting B's vruntime to `min(vruntime of tree)` on wake-up. Small fairness cost for jobs that sleep often.

## How They Handle New Jobs

| Scheduler | New-job treatment |
|---|---|
| Lottery | Just update total ticket count — no global state to reconcile. |
| Stride | Non-trivial: what pass value? Affects fairness immediately. |
| CFS | Set vruntime to tree minimum — same trick as for sleepers. |

## Problems Shared by Proportional-Share

- **Ticket / nice assignment is open** — how many tickets should your text editor get? The scheduler can't decide.
- **I/O handling is awkward** — interactive jobs that sleep frequently don't always get their full share (as seen in CFS's sleep-wake cutoff).
- **Less friendly for general-purpose workloads** than adaptive schedulers like [[MLFQ]] that just learn from behavior.

## Where Proportional Share Shines

- **Virtualization / cloud** — "give this VM 25% of CPU, reliably". Proportional-share is a natural fit.
- **Multi-tenant datacenters** — guarantee fractions to paying customers.
- **Real-time-ish** applications where bounded CPU share matters more than best turnaround.

## Tip — Use Efficient Data Structures When Appropriate

> [!tip] Hallmark of good engineering
> Simple lists don't scale to thousands of processes. CFS's red-black tree lowered scheduler overhead enough to matter at datacenter scale (where the scheduler alone consumes ~5% of CPU in modern studies).

## Related Notes

- [[Proportional Share Scheduling]] — the family overview.
- [[Lottery Scheduling]], [[Stride Scheduling]], [[CFS - Completely Fair Scheduler]].
- [[MLFQ]] — the general-purpose alternative.
- [[Scheduler]], [[Scheduling Metrics]].
- [[MOC - CPU Virtualization]].
