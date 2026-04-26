---
title: Inverted Page Table
tags:
  - concept
  - ostep
  - memory
  - paging
type: concept
introduced_in: Ch 20
---

# Inverted Page Table

> [!abstract] Definition
> An **inverted page table** is a single global table with **one entry per physical frame**, recording which process and which [[Page|virtual page]] (if any) currently occupies that frame. Lookup is by search — typically a hash table over (ASID, VPN). Total size depends on **physical RAM**, not on how many or how big the address spaces are.

## The Inversion

| Linear / Multi-Level | Inverted |
|---|---|
| One table per process | One global table |
| Indexed by **VPN** | Indexed by **PFN** |
| Stores PFN | Stores ASID + VPN |
| Lookup is direct (array index) | Lookup is search (hash) |
| Size grows with address space | Size grows with physical memory |

## Why It Saves Memory

For a 64-bit address space, a linear table is impossible (32 PB) and even multi-level is non-trivial. An inverted table has size proportional to RAM:

- 64 GB RAM, 4 KB pages → 16 M frames → 16 M entries × 16 bytes ≈ **256 MB total** for the entire system, regardless of how many processes run.

Compare: 1000 processes × 4 MB linear page tables = 4 GB just for tables.

## The Catch: Lookup Cost

To translate VPN `X` for the current process:
1. Compute `hash(ASID, X)`.
2. Probe the hash table — possibly with collisions.
3. If found, the entry's index is the PFN.
4. If not found, the page isn't resident → page fault.

Lookup is O(1) average for hashing, but constants and cache behavior are worse than an indexed array. Hash collisions create variable latency.

## Sharing Is Awkward

A single physical frame may be shared by N processes. With one entry per frame, you can't represent "shared by P1 VPN 10 and P2 VPN 50" in one slot. Workarounds: chained shared lists, separate sharing maps. Adds complexity.

## Where It's Used

- **PowerPC** — historically used a hardware-managed hashed inverted page table.
- **HP PA-RISC** — similar.
- **IA-64 Itanium** — supported both.
- **x86, ARM, RISC-V** — all chose multi-level instead.

Linux on PowerPC translates the inverted hardware table into its multi-level software model — meaning even on inverted-PT hardware, the OS often works with multi-level conceptually.

## Trade-Offs Summary

| Pro | Con |
|---|---|
| Memory ∝ RAM, not VA size | Lookup is search, not index |
| Excellent for huge sparse VAs | Sharing complications |
| Good fit for software-managed TLB | Cache-unfriendly lookups |

## Related Notes

- [[Multi-Level Page Table]] — the dominant alternative.
- [[Page Table]] — the umbrella concept.
- [[ASID]] — needed in entries to distinguish processes.
- [[TLB]] — what avoids most of the lookup cost.
- [[Hardware-Managed vs Software-Managed TLB]] — software-managed naturally pairs with inverted.
- [[Sparse Address Space]] — what inverted page tables handle elegantly.
- [[Paging]] — the mechanism.
- [[Ch 20 — Paging - Smaller Tables]].
