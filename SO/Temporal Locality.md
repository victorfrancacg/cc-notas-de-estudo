---
title: Temporal Locality
tags:
  - concept
  - ostep
  - cache
  - locality
type: concept
introduced_in: Ch 19
---

# Temporal Locality

> [!abstract] Definition
> **Temporal locality** is the empirical principle that if a program accesses a memory location `x`, it is likely to access `x` **again soon**. Caches exploit this by **keeping** recently-accessed data: a re-access becomes a cheap hit instead of an expensive miss.

## Why It Exists

- **Loop variables** — `i` and `j` are read every iteration.
- **Hot functions** — the same code path runs over and over.
- **Working set** — short windows of execution touch only a small subset of the program's data.
- **Repeatedly-traversed data structures** — hash tables, trees, queues.

## In the TLB

A loop running for thousands of iterations re-uses the same handful of pages → the same TLB entries → near-100% hit rate after warmup.

## In CPU Caches

The same hot variable lives in L1d for as long as it's being accessed. The cost of bringing it in is paid once; thousands of subsequent accesses hit.

## In Replacement Policies

The reason **LRU** (least-recently-used) eviction is a strong default is that it bets on temporal locality: data accessed recently is more likely to be accessed again than data that hasn't been touched in a while. Approximate LRU (clock, NRU) drives the [[Page Replacement]] decisions in real OSes.

## Failure Modes

Temporal locality breaks for:

- **Streaming workloads** — reading a 100 GB log file once, never again. No reason to cache; eviction policy should ideally **bypass** the cache.
- **Loops that overflow the cache** — if your hot data is just larger than the cache, every iteration re-misses everything.
- **Cache scans** — scanning the cache itself (uncommon, but happens in some access patterns) defeats locality.

Modern systems offer cache-bypass / non-temporal hints (`movnt` on x86, `__builtin_prefetch`) for streaming workloads.

## Related Notes

- [[Spatial Locality]] — the sibling principle.
- [[TLB]] — exploits both forms of locality.
- [[Page Replacement]] — based on the temporal-locality assumption.
- [[Ch 19 — Paging - Faster Translations (TLBs)]].
