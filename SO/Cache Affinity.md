---
title: Cache Affinity
tags:
  - concept
  - ostep
  - scheduling
  - multiprocessor
  - performance
type: concept
introduced_in: Ch 10
---

# Cache Affinity

> [!abstract] Definition
> **Cache affinity** is the tendency for a process to run faster on the same CPU it last ran on — because its data and instructions are already cached there (L1, L2 caches + TLB entries + branch predictor state). Multiprocessor schedulers try to preserve this affinity by keeping processes on their "home" CPU when possible.

## Why It Matters

Cold caches cost **hundreds to thousands** of cycles of misses as a process re-populates them. On a machine where an instruction takes one cycle, even a handful of misses is an eternity.

Every time a process migrates from CPU_i to CPU_j:

- The useful data in CPU_i's caches goes to waste.
- CPU_j's caches must be re-populated from main memory.
- Until warmed, the process runs slower than it would have on CPU_i.

## What "Affinity State" Includes

Not just data caches. When a process runs on a specific CPU, it builds up **local state** in:

| Structure | What's cached |
|---|---|
| **L1 / L2 / L3 cache** | Hot memory lines from recent accesses. |
| **TLB** | Virtual→physical translations for the process. |
| **Branch predictor** | Histories of branches taken. |
| **Prefetcher state** | Learned access patterns. |

All of this evaporates (or becomes inapplicable) on migration.

## Affinity vs Load Balance — The Tension

> [!warning] Two goals pull opposite ways
> - **Keep processes home** — good for cache hit rates.
> - **Balance load across CPUs** — good for utilization.
>
> If one CPU is overloaded and another is idle, affinity says "leave them alone", load balance says "move something". Real schedulers compromise based on thresholds.

## How Different Scheduler Architectures Handle It

| | How affinity behaves |
|---|---|
| [[Single-Queue Multiprocessor Scheduling (SQMS)\|SQMS]] | Poor by default — jobs bounce around. Needs affinity hints. |
| [[Multi-Queue Multiprocessor Scheduling (MQMS)\|MQMS]] | **Great by default** — jobs stay on their home CPU. Migration only when load-balancing. |

## Programmer Control

UNIX lets programs pin themselves:

- `sched_setaffinity(pid, cpuset)` (Linux) — force a process to only use certain CPUs.
- `taskset -c 2 ./my_program` (shell) — pin at launch.

Useful for latency-sensitive workloads, benchmarks, or isolating noisy neighbors.

## Super-Linear Speedup

A fun phenomenon: splitting a job across N CPUs can yield *more* than N× speedup, because the working set per CPU shrinks to fit cache. OSTEP's Ch 10 homework explores this.

## Related Notes

- [[Cache Coherence]] — the correctness sibling concept.
- [[Context Switch]] — also has cache-cooling costs, even on one CPU.
- [[Single-Queue Multiprocessor Scheduling (SQMS)]], [[Multi-Queue Multiprocessor Scheduling (MQMS)]].
- [[Load Balancing (Scheduling)]].
- [[Ch 10 — Multiprocessor Scheduling]].
