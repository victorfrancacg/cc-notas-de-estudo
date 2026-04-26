---
title: Memory Hierarchy
tags:
  - concept
  - ostep
  - memory
  - architecture
type: concept
introduced_in: Ch 21
---

# Memory Hierarchy

> [!abstract] Definition
> The **memory hierarchy** is the layered organization of storage in a computer system, ordered by speed (fastest → slowest) and inversely by capacity (smallest → largest). Each layer caches a subset of the next-slower layer's contents. [[Paging|Virtual memory]] uses RAM as a cache for disk-backed storage; the [[TLB]] is a cache for [[Page Table|page-table]] entries; CPU caches are caches for RAM.

## The Tiers

| Tier | Size | Latency | What it caches |
|---|---|---|---|
| **CPU registers** | ~1 KB | <1 ns | hot variables |
| **L1 cache** | 32–64 KB | ~1 ns | L2 contents |
| **L2 cache** | 256 KB – 1 MB | ~3 ns | L3 contents |
| **L3 cache** | 8–64 MB | ~10 ns | RAM contents |
| **RAM (DRAM)** | GBs | ~100 ns | disk contents (via VM) |
| **NVMe SSD** | TBs | ~100 µs | data and swap |
| **HDD** | TBs–PBs | ~10 ms | bulk data |
| **Network / cloud** | unbounded | ms–s | remote data |

Each step down: ~10–1000× slower, ~10–1000× larger. The economics of fast vs cheap force the hierarchy.

## Caching Up the Hierarchy

Every level except the bottom is a **cache** for the level below. The same caching tricks apply at every layer:

- **Locality** ([[Spatial Locality|spatial]] and [[Temporal Locality|temporal]]) — programs reference a small subset over short windows.
- **Replacement** — when the cache is full, evict something. Each layer has its own [[Page Replacement|policy]].
- **Coherence** — keep multiple cached copies consistent (hardware does this for L1/L2/L3; OS does it via writeback for RAM↔disk).

## Where Virtual Memory Sits

[[Memory Virtualization|Virtual memory]] makes RAM a cache for disk-backed storage:

- **Hits**: page is resident → ~100 ns load.
- **Misses**: [[Page Fault|page fault]] → fetch from disk → 10⁴–10⁵× slower.

The vast latency gap is why [[Page Replacement|replacement policy]] matters so much for VM (Ch 22) compared to L1/L2 (where misses cost only 10×).

## "Faster, Smaller, More Expensive" Trade-Off

Why not just use only registers (fastest)? Or only RAM (cheap, large)?

- Smaller, faster storage **costs more per byte** (silicon area, power).
- Larger, slower storage costs less but can't keep up with the CPU.

The hierarchy lets you have the *illusion* of a single big fast memory: hot data lives in caches, cold data lives in DRAM/disk, the system shuffles transparently.

## Implications for Programmers

- **Working set < L1** → blazing fast.
- **Working set < L3** → fast.
- **Working set < RAM** → reasonable.
- **Working set > RAM** → falling off a cliff (10⁴× slowdown).

Cache-aware algorithm design (blocking, tiling, struct-of-arrays) explicitly courts the smaller tiers.

## Related Notes

- [[TLB]] — caches translations within the MMU.
- [[Spatial Locality]], [[Temporal Locality]] — why caching works.
- [[Swap Space]] — the disk tier under RAM in VM systems.
- [[Page Fault]] — the cost of a RAM-cache miss.
- [[Page Replacement]] — the policy at the RAM-cache boundary.
- [[Memory Virtualization]] — uses the hierarchy explicitly.
- [[Ch 21 — Beyond Physical Memory - Mechanisms]].
