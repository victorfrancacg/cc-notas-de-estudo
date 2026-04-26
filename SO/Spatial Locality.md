---
title: Spatial Locality
tags:
  - concept
  - ostep
  - cache
  - locality
type: concept
introduced_in: Ch 19
---

# Spatial Locality

> [!abstract] Definition
> **Spatial locality** is the empirical principle that if a program accesses memory at address `x`, it is likely to soon access an address **near** `x`. This is what makes caches (CPU L1/L2/L3, [[TLB]], page cache) effective: bringing in a chunk of nearby data on each miss amortizes the miss cost over many subsequent hits.

## Why It Exists

Programs are not random:

- **Arrays** are walked sequentially.
- **Code** runs straight-line most of the time (one instruction after the next).
- **Structs** access neighboring fields.
- **Stack frames** are contiguous.

So accesses cluster spatially. Hardware exploits this by fetching a **cache line** (64 bytes typical) on every miss, not just the requested byte.

## In the TLB

A 4 KB page holds 1024 ints. A loop walking an int array touches one TLB entry for 1024 consecutive accesses. The first access on each page misses; the next 1023 hit — purely spatial locality.

## In CPU Caches

L1d caches load full 64-byte lines. Access `a[0]` → line containing `a[0..15]` (assuming 4-byte ints) is now cached. `a[1]` through `a[15]` hit for free.

## In the Page Cache

When a process touches a file via `read()` or `mmap()`, the kernel often **read-aheads** (prefetches) the next several pages — betting on spatial locality.

## Failure Modes

Spatial locality breaks for:

- **Random pointer chasing** (linked lists, hash tables with poor hashing) — each access goes somewhere unrelated.
- **Strided access at cache-unfriendly stride** — e.g., walking columns of a row-major matrix.
- **Hot/cold data interleaved** — accessing one field of a struct repeatedly when the struct is large; cache lines bring in unused bytes.

Software techniques (struct-of-arrays, cache blocking, prefetching hints) exist to combat these — see any "performance engineering" treatment.

## Related Notes

- [[Temporal Locality]] — the time-based sibling.
- [[TLB]] — exploits spatial locality across page boundaries.
- [[Ch 19 — Paging - Faster Translations (TLBs)]].
