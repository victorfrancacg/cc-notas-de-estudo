---
title: Working Set
tags:
  - concept
  - ostep
  - memory
  - paging
  - policy
aliases:
  - Working Set Model
  - WSS
  - Resident Set Size
  - RSS
type: concept
introduced_in: Ch 22
---

# Working Set

> [!abstract] Definition
> A process's **working set** at time $t$, written $W(t, \tau)$, is the set of [[Page|pages]] referenced in the time window $[t-\tau, t]$. Denning's working-set model (1968) says that to run efficiently, a process needs at least its working set resident in physical memory; otherwise it [[Thrashing|thrashes]]. The OS uses this idea for **admission control** and per-process **resident-set limits**.

## The Idea

Programs have phases. During each phase, they touch a small subset of their address space — the working set. Working sets evolve over time but at any instant are concentrated.

```
    pages
    touched
       │           ┌──┐         ┌────┐
       │   ┌─┐    │  │    ┌────┘    │
       │  │ └─┐  │  └────┘          └─
       └────────────────────────────── time
        phase 1   phase 2     phase 3
```

If RAM holds the working set, page faults are rare (cold starts only) → fast. If RAM is smaller than the working set, every reference can fault → catastrophic slowdown.

## Resident Set Size (RSS)

The actual count of a process's pages currently in physical memory. Often used as a proxy for the working set:

- `top`, `ps`, `htop` show RSS as the "real memory used" column.
- Linux's `/proc/<pid>/status` reports `VmRSS` (resident bytes).

RSS ≤ working-set-size means the process is "thin" — under-resourced. RSS = WSS means happy.

## Admission Control

If the **sum** of all running processes' working sets exceeds RAM, the system thrashes. Some classical OSes (and modern HPC schedulers) **admission-control** new processes:

- Estimate working-set sizes.
- Refuse to start a new process whose addition would push total > RAM.
- Better to have N happy processes than 2N thrashing.

Linux mostly doesn't admit-control — it runs everything and **kills** memory hogs (OOM killer) when things go wrong.

## Per-Process Limits in VAX/VMS

VAX/VMS used a **segmented FIFO** with a per-process **resident set size (RSS)**. When the process exceeded its quota, its oldest page was evicted — *its own*, not anyone else's. Local replacement, not global. Made the system fairer but slightly less efficient.

## How Long Is the Window τ?

There is no universal answer. τ in seconds matches what most kernels use (page-aging epochs). Some systems use **virtual time** (instructions executed) instead of wall time, so processes that don't run don't lose pages.

In practice, modern Linux doesn't compute formal working sets — it uses the [[Clock Algorithm|two-list active/inactive scheme]] that approximates the same effect.

## Why It Matters

> [!tip] Working-set sizing is capacity planning
> Database tuning ("buffer pool"), JVM heap sizing, container memory limits — all are essentially asking "what is this workload's working set?" Get it right, performance is great. Too small → faults dominate.

## Related Notes

- [[Thrashing]] — what happens when working sets don't fit.
- [[Page Replacement]] — the policy that tries to keep the WS resident.
- [[LRU]] — its decisions tend to keep the working set in memory.
- [[Demand Paging]] — relies on locality, which the WS model formalizes.
- [[Spatial Locality]], [[Temporal Locality]] — the empirical basis.
- [[Ch 22 — Beyond Physical Memory - Policies]].
