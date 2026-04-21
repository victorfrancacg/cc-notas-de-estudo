---
title: Cache Coherence
tags:
  - concept
  - ostep
  - multiprocessor
  - hardware
  - architecture
type: concept
introduced_in: Ch 10
---

# Cache Coherence

> [!abstract] Definition
> **Cache coherence** is the hardware guarantee that, on a multiprocessor system, writes to shared memory are eventually visible to all CPUs, despite each CPU having its own private cache. Without it, one CPU could write a value and another CPU could indefinitely read the stale cached copy.

## The Problem

```
CPU1 cache: A = 10
CPU2 cache: A = 10

CPU1 writes A = 20
  CPU1 cache: A = 20
  Main memory: (eventually) A = 20
  CPU2 cache: still 10        <-- BUG if nothing corrects this
```

Without coherence, CPU2 reading `A` keeps getting 10 forever.

## The Solution — Bus Snooping

On bus-based systems, every cache watches the bus. When a CPU writes `A`:

1. The write (or its intent) is visible on the bus.
2. Other caches holding `A` **either invalidate** their copy (MESI invalidate protocol) **or update** it to the new value (update protocol).
3. Next time a snooping CPU reads `A`, its cache misses (or sees the new value), pulling the correct data.

Modern CPUs use more sophisticated variants (MESI, MOESI, MESIF) and directory-based coherence for non-bus topologies (NUMA), but the core idea is the same.

## What Coherence Does NOT Give You

> [!warning] Coherence ≠ atomicity
> Coherence ensures every write is *eventually* seen by other CPUs. It does **not** prevent races — if thread A and thread B both read `A=10`, both increment, both write `A=11`, coherence happily distributes `A=11` to both, and you've lost an increment.
>
> This is why OS code needs **locks**, **atomic instructions**, and **memory fences** on top of cache coherence. See [[Concurrency]].

## The Trade-Off

Cache coherence makes the programmer's life much easier — you can mostly pretend shared memory "just works". But it costs hardware complexity and traffic on every write to shared data. On high-core-count machines, coherence traffic is a real scaling bottleneck.

## Related Notes

- [[Cache Affinity]] — coherence's performance sibling.
- [[Concurrency]] — why coherence alone isn't enough.
- [[Ch 10 — Multiprocessor Scheduling]].
