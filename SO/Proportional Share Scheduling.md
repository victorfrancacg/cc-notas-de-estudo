---
title: Proportional Share Scheduling
tags:
  - concept
  - ostep
  - scheduling
aliases:
  - Fair-Share Scheduling
  - Proportional-Share Scheduler
type: concept
introduced_in: Ch 9
---

# Proportional Share Scheduling

> [!abstract] Definition
> A **proportional-share** (a.k.a. **fair-share**) scheduler is one whose primary goal is to give each job a **specified fraction** of CPU, rather than minimize turnaround or response time. If A's share is 75% and B's is 25%, over any meaningful interval A should get roughly 3× B's CPU time.

## What Makes It Different

| Traditional scheduler (MLFQ, RR) | Proportional-share scheduler |
|---|---|
| Optimizes turnaround or response. | Honors CPU share contracts. |
| Adjusts based on observed behavior. | Respects externally assigned priorities. |
| Answer to "who runs next?": "whoever has best metric now". | Answer: "whoever is most behind its share". |

## Canonical Members

- **[[Lottery Scheduling]]** — random ticket draw.
- **[[Stride Scheduling]]** — deterministic, each job advances at a stride inversely proportional to share.
- **[[CFS - Completely Fair Scheduler|CFS]]** — Linux's modern fair scheduler using virtual runtime and a red-black tree.

## Why You'd Want It

- **Cloud / virtualization** — "give tenant X 25% of CPU, guaranteed".
- **Multi-user systems** — fair sharing across users / groups regardless of process count.
- **SLA-driven services** — contractual CPU guarantees.

## Where It's a Poor Fit

- **Desktop / interactive workloads** — what users want is *snappy responses to clicks and typing*, not strict fractions. [[MLFQ]] or [[MLFQ]]-like schedulers feel better here.
- **Batch workloads with unknown priorities** — nobody knows how many tickets to assign.

## The Ticket / Share Assignment Problem

> [!warning] Who decides shares?
> Proportional-share schedulers **assume** the shares are given — but nothing in the scheduler says **how** to pick them. In practice: admins, contracts, or a layer above.
>
> On a desktop, what share should your editor get? Your compiler? Nobody has a principled answer.

This is a real limitation — it's why general-purpose OSes that use proportional-share (like Linux with CFS) still *default* everyone to equal shares, with `nice` as the only user-visible knob.

## The I/O Problem

All three proportional-share algorithms struggle with processes that frequently sleep (for I/O or interactivity):

- Lottery: sleeping jobs just don't win lotteries — they lose their share.
- Stride: sleeping jobs' pass values stagnate; on wake they could monopolize to catch up.
- CFS: needs a special rule to handle wake-up vruntime (otherwise starves long-running jobs).

None of them handle I/O as naturally as MLFQ does.

## Related Notes

- [[Lottery Scheduling]], [[Stride Scheduling]], [[CFS - Completely Fair Scheduler]].
- [[MLFQ]] — contrasting family (adaptive, behavior-driven).
- [[Scheduler]], [[Scheduling Metrics]].
- [[Ch 9 — Scheduling - Proportional Share]].
