---
title: Page
tags:
  - concept
  - ostep
  - memory
  - paging
aliases:
  - Virtual Page
  - VP
type: concept
introduced_in: Ch 18
---

# Page

> [!abstract] Definition
> A **page** is a fixed-size region of a process's [[Address Space|virtual address space]] under [[Paging]]. Typical size: 4 KB. Pages are the unit of mapping — each page is independently mapped to a [[Page Frame|physical frame]] by the [[Page Table]].

## Page Size

- **4 KB** — default on x86, ARM, RISC-V.
- **2 MB / 1 GB** — "huge pages", explicitly requested. Fewer [[TLB]] misses for large allocations.
- **16 KB** on Apple Silicon (some configs).

The page size is a fundamental architectural constant; the OS and hardware must agree.

## Virtual Page Number (VPN)

A virtual address splits into:

```
+------+--------+
| VPN  | offset |
+------+--------+
```

- **VPN** — selects which page in the address space.
- **offset** — selects a byte within that page.

For 4 KB pages, the offset is 12 bits (2¹² = 4096) and the VPN is the rest of the address.

## Pages vs Frames

| Page (virtual) | Frame (physical) |
|---|---|
| Per-process; many | Global; one set |
| Identified by VPN | Identified by PFN |
| Mapped via [[Page Table]] | Holds page data |

A virtual page can be:
- **Mapped** to a frame (resident in RAM).
- **Swapped** to disk (not resident; trap on access — see [[Page Fault]]).
- **Unmapped** (invalid VPN — trap kills the process — see [[Segmentation Fault]]).
- **Shared** (multiple processes' VPNs map to the same PFN).

## Page Lifecycle

1. **Allocate** — process requests memory; OS reserves a virtual page (often without yet allocating a frame).
2. **First access** → page fault → OS allocates a physical frame, zeros it (for anonymous), or reads from disk (for file-backed).
3. **Resident** — translation cached in [[TLB]] or walked via page table.
4. **Maybe swapped out** — OS may evict the page to disk under memory pressure.
5. **Free** — process exits or `munmap()` releases the page.

## Why Fixed Size

Variable-size units ([[Segmentation|segments]]) suffer [[External Fragmentation|external fragmentation]]. Fixed-size pages mean any frame fits any page, with no fit search and no compaction. The cost is [[Internal Fragmentation|internal fragmentation]] — at most one partial page per allocation.

## Related Notes

- [[Page Frame]] — the physical-side counterpart.
- [[Page Table]] — maps VPN → PFN.
- [[Paging]] — the umbrella scheme.
- [[TLB]] — caches recent VPN → PFN translations.
- [[Page Fault]] — what happens when a page isn't resident.
- [[Virtual Address]] — split into VPN + offset.
- [[Ch 18 — Paging - Introduction]].
