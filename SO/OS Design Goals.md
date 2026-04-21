---
title: OS Design Goals
tags:
  - concept
  - ostep
  - design-principle
type: concept
introduced_in: Ch 2
---

# OS Design Goals

> [!abstract] TL;DR
> OS design is the art of juggling several goals that pull in different directions. No OS maximizes all of them; every design is a point in a tradeoff space.

## The Canonical List (OSTEP §2.5)

| Goal | What it means | Enemy of… |
|---|---|---|
| **Abstraction** | Hide messy hardware behind clean interfaces. Foundation of CS. | Raw performance (every layer adds cost). |
| **Performance** | Minimize overheads — both time (CPU cycles) and space (memory). | Abstraction, safety checks, isolation. |
| **Protection / [[Isolation (OS)\|Isolation]]** | A misbehaving app can't damage another app or the OS. | Raw performance, simplicity. |
| **Reliability** | The OS must not crash; if it does, everything crashes with it. | Complexity, aggressive features. |
| **Energy efficiency** | Especially critical on mobile/embedded. | Always-on services, polling. |
| **Security** | Resist malicious apps, not just buggy ones. | Convenience, performance. |
| **Mobility** | Run on small/constrained devices. | Feature richness. |

## The Fundamental Tensions

> [!warning] You can't have it all
> Abstractions help programmers but cost cycles. Isolation helps reliability but requires protection mechanisms (traps, MMU checks). Performance and safety are the classic adversaries.

## How to Read the Rest of OSTEP

Every mechanism and policy you'll meet is a specific choice on this tradeoff space:

- [[Paging]] vs [[Segmentation]] — flexibility vs overhead.
- [[Preemptive vs Cooperative Scheduling]] — responsiveness vs simplicity.
- [[Journaling]] — reliability bought with write overhead.
- [[TLB]] — performance added on top of a slow mechanism ([[Page Table]] walks).

## Related Notes

- [[Mechanism vs Policy]].
- [[Isolation (OS)]].
- [[Operating System]].
- [[Principle of Least Privilege]].
