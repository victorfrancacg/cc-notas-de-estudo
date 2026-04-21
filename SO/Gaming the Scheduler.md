---
title: Gaming the Scheduler
tags:
  - concept
  - ostep
  - scheduling
  - pitfall
  - security
type: concept
introduced_in: Ch 8
---

# Gaming the Scheduler

> [!abstract] Definition
> **Gaming the scheduler** means deliberately structuring a program's behavior so the scheduler grants it more than its fair share of CPU. It's a class of attack/exploit born from **scheduling policy blind spots** — the scheduler can't tell "I'm being clever" from "I'm a legit interactive job".

## The Classic [[MLFQ]] Exploit

Under an early MLFQ rule set (pre-OSTEP's final version):

- **Rule 4a:** Job uses up slice → priority reduced.
- **Rule 4b:** Job gives up CPU before slice ends → priority **stays the same**.

A clever attacker writes:

```c
while (true) {
    burn_CPU_for(allotment_minus_epsilon);
    issue_trivial_io();        // e.g., write one byte
}
```

The trivial I/O triggers Rule 4b — the process stays at the top priority. It monopolizes up to 99% of CPU while impersonating an interactive job.

## The Fix

Track CPU usage **cumulatively** at each priority level, ignoring how many times the process voluntarily yielded.

> Rule 4 (final): Once a job uses up its time allotment at a level — regardless of how many times it gave up the CPU — its priority is reduced.

Now the attacker's 99-ms busy loop counts toward the allotment even though it was split across many "yield + I/O" calls.

## Why It's a Security Concern

> [!warning] Scheduling is a security surface
> In a shared datacenter, one tenant gaming the scheduler can:
> - Steal CPU from other tenants — a resource-theft attack.
> - Cause **noisy-neighbor** problems that look like random latency to innocent users.
> - In extreme cases, influence timing-sensitive code paths (side channels).
>
> This is why scheduling policy design is not just about performance — it's also about **fairness enforcement** across untrusted actors.

## Other Scheduler Exploits

- **Nice-value abuse** (less a hack, more an oversight) — if the admin forgets to cap user nice values, a user can self-renice to higher priority.
- **Fork-bomb** — not quite "gaming", but a denial-of-service that overwhelms the scheduler by creating thousands of processes.
- **Priority inversion attacks** — binding a high-priority task to a lock held by a low-priority attacker can stall the high-priority task. (Mars Pathfinder famously hit this.)

## Generalizable Lesson

Any policy that makes decisions based on **observable proxies** (e.g., "did you yield the CPU?") can be gamed by controlling those proxies. Robust policies either:

- Measure **cumulative** or long-term behavior (harder to fake).
- Require **credentials / privileges** to enter favored states.
- Randomize choices (as [[Lottery Scheduling]] does).

## Related Notes

- [[MLFQ]] — where this exploit was classically found and fixed.
- [[Lottery Scheduling]] — randomness as an anti-gaming mechanism.
- [[Starvation (Scheduling)]] — cousin problem.
- [[Isolation (OS)]], [[OS Design Goals]].
