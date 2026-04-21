---
title: Mechanism vs Policy
tags:
  - concept
  - ostep
  - design-principle
aliases:
  - Separation of Mechanism and Policy
  - Mechanism and Policy
type: concept
introduced_in: Ch 2
---

# Mechanism vs Policy

> [!abstract] Definition
> - **Mechanism** — the low-level *how*: the machinery that *can* do something (e.g., how to switch the CPU from one process to another).
> - **Policy** — the high-level *which / when*: the decision about *what to do* using the mechanism (e.g., which process to run next).
>
> Good OS design keeps these **separate**, so you can change one without rewriting the other.

## Why Separate Them?

1. **Reusability.** One solid mechanism supports many policies (swap in a new scheduler without rewriting the context switch code).
2. **Clarity.** Arguments about fairness and throughput live in the policy; arguments about correctness and safety live in the mechanism.
3. **Testability.** Mechanism correctness can be verified once, forever; policies can be experimented with.

## Canonical Examples in OSTEP

| Area | Mechanism | Policy |
|---|---|---|
| CPU scheduling | [[Context Switch]], [[Timer Interrupt]], [[Trap]] | [[FIFO Scheduling]], [[Round Robin]], [[MLFQ]], [[CFS - Completely Fair Scheduler]] |
| Memory | [[Address Translation]] (base/bounds, paging, TLB) | Page replacement: [[LRU]], [[FIFO Replacement]], [[Clock Algorithm]] |
| File system | Raw block I/O, inode layout | Allocation policy, caching policy |

## The Slogan

> [!tip] Mechanism answers "how?" Policy answers "which?"
> If you catch yourself building machinery that only works for one particular scheduling rule, you've smuggled policy into mechanism.

## Careful — The Boundary Isn't Always Clean

In practice, some policies demand mechanism support (e.g., [[MLFQ]]'s priority boost requires per-job accounting machinery). Clean separation is a *design ideal*, not an inviolable law.

## Related Notes

- [[Separation of Concerns (OS Design)]].
- [[Scheduler]] — the embodiment of CPU policy.
- [[Context Switch]] — the embodiment of CPU mechanism.
- [[OS Design Goals]].
