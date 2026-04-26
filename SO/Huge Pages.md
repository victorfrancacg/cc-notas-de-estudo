---
title: Huge Pages
tags:
  - concept
  - ostep
  - memory
  - paging
  - performance
aliases:
  - Large Pages
  - Superpages
  - Transparent Huge Pages
  - THP
type: concept
introduced_in: Ch 23
---

# Huge Pages

> [!abstract] Definition
> **Huge pages** are virtual-memory [[Page|pages]] larger than the default 4 KB — typically **2 MB** or **1 GB** on x86. They reduce [[TLB]] pressure (each TLB entry covers more memory) and shrink [[Page Table|page tables]]. Modern OSes support them either explicitly (apps opt in) or transparently (kernel promotes contiguous regions automatically).

## The Motivation: TLB Coverage

A 64-entry TLB with 4 KB pages covers **256 KB**. Programs with multi-MB hot data sets exceed coverage and TLB-miss constantly:

```
TLB coverage = (entries × page_size)
   64 × 4 KB  =   256 KB    ← typical L1 dTLB
   64 × 2 MB  =   128 MB    ← with huge pages: 512× coverage
   64 × 1 GB  =    64 GB    ← with gigantic pages
```

Workloads where this matters:
- **Database buffer pools** (multi-GB).
- **JVM / GC heaps** (10s of GB).
- **ML model weights** (100s of GB).
- **Scientific simulations** with large arrays.

OSTEP cites measurements: some applications spend ~10% of cycles servicing TLB misses with 4 KB pages — almost eliminated with 2 MB pages.

## Page-Table Reduction

Bigger pages → fewer PTEs:

- 32-bit address space, 4 KB pages → 1 M PTEs → 4 MB page table.
- 32-bit address space, 4 MB pages → 1024 PTEs → 4 KB page table.

For huge address spaces, this saves real memory.

## Trade-Offs

> [!warning] [[Internal Fragmentation]]
> A 2 MB huge page allocated for a 100 KB working buffer wastes 1.9 MB. Sparse-large workloads (huge VAs with scattered hot regions) bleed memory.

> [!warning] Allocation requires contiguity
> A 2 MB huge page needs **2 MB of contiguous physical memory**. Under fragmentation, this can fail or trigger expensive **memory compaction**.

> [!warning] Compaction stalls
> Linux's transparent huge pages run a background "khugepaged" thread that defragments memory to find contiguous regions. Sometimes it stalls latency-critical work; some production environments disable THP.

## Linux Modes

### Explicit Huge Pages (HugeTLB)

```c
mmap(NULL, size, PROT_READ|PROT_WRITE,
     MAP_PRIVATE|MAP_ANONYMOUS|MAP_HUGETLB, -1, 0);
```

- Apps opt in.
- Page size selectable (`MAP_HUGE_2MB`, `MAP_HUGE_1GB`).
- The kernel maintains a separate huge-page pool (configured at boot).
- **Predictable** — no compaction stalls.

Used by databases (Oracle, PostgreSQL with `huge_pages=on`), DPDK, JVM (`-XX:+UseLargePages`).

### Transparent Huge Pages (THP)

- Kernel automatically promotes regions of 4 KB pages to 2 MB when contiguous.
- No app changes needed.
- Background `khugepaged` performs the promotions.

Modes:
- `always` — aggressive; default in many distros.
- `madvise` — only when apps hint via `madvise(MADV_HUGEPAGE)`.
- `never` — disabled.

## When To Use, When Not

> [!tip] Workloads that benefit
> - Large in-memory data structures touched randomly (databases, hash tables, ML).
> - Long-lived servers where the cost of contiguity is amortized.

> [!warning] Workloads that suffer
> - Latency-critical apps that can't tolerate khugepaged stalls.
> - Sparse workloads where huge-page granularity wastes memory.
> - Short-lived processes (the contiguity setup cost outweighs the TLB savings).

A common production pattern: **disable THP, enable explicit HugeTLB** for the few apps (databases) that benefit, leaving the rest with predictable 4 KB pages.

## Hardware Support

- **x86_64**: 4 KB, 2 MB, 1 GB.
- **ARM64**: 4 KB, 16 KB, 64 KB base; plus 2 MB / 32 MB / 1 GB / 16 GB huge variants.
- **RISC-V**: similar tiered structure.

## Related Notes

- [[TLB]] — what huge pages relieve.
- [[Page]], [[Page Frame]] — units of paging.
- [[Page Table]], [[Multi-Level Page Table]] — what huge pages shrink.
- [[Internal Fragmentation]] — the cost.
- [[Linux Virtual Memory]] — implementation.
- [[mmap()]] — how apps request explicit huge pages.
- [[Ch 23 — Complete Virtual Memory Systems]].
