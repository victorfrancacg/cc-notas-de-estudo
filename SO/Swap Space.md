---
title: Swap Space
tags:
  - concept
  - ostep
  - memory
  - disk
aliases:
  - Swap Partition
  - Swap File
  - Pagefile
type: concept
introduced_in: Ch 21
---

# Swap Space

> [!abstract] Definition
> **Swap space** is a region of disk reserved by the OS for temporarily holding [[Page|virtual pages]] evicted from physical memory. When an evicted page is later accessed, the [[Page-Fault Handler|page-fault handler]] reads it back. Swap space's size sets the **upper bound** on how much memory all running processes can collectively appear to use.

## What Goes to Swap

| Page type | Swappable? | Where it comes from on fault |
|---|---|---|
| Anonymous (heap, stack, BSS) | yes — to swap | swap space |
| File-backed (mmap'd file) | no — written back to file | the file |
| Code (executable text) | no | re-read from binary |
| Read-only data | no | re-read from binary |

Read-only and file-backed pages don't need swap because their contents are already on disk somewhere. Only **anonymous, modified** pages need swap.

## Forms of Swap

- **Swap partition** — a dedicated disk partition (Linux convention). Contiguous, fastest.
- **Swap file** — a regular file used as swap (Windows `pagefile.sys`, Linux optional). More flexible (resizable), slightly slower.
- **Multiple swap devices** — Linux supports multiple, with priority-ordered allocation.

## Swap Layout

The OS maintains, per evicted page, the **disk block number** in swap holding it. This is stored *inside the PTE itself* — the `PFN` field is repurposed as a swap-block pointer when `Present=0`. Same bits, different meaning depending on the present bit.

## Swap Space ≠ "Slow RAM"

It's important to note: swap is **fundamentally different** from RAM, not just slower:

- Access latency: ~10 ms for HDD swap, ~100 µs for SSD swap, ~100 ns for RAM. Even fast SSDs are 1000× slower than RAM.
- Bandwidth: comparable for sequential, much lower for random.
- Wear: SSDs have limited write cycles; aggressive swapping shortens life.

> [!warning] "Thrashing" — swap's nightmare
> When the working set exceeds RAM and the OS spends all its time swapping pages in/out instead of running user code, the system is **thrashing**. Symptoms: very high disk I/O, very low CPU utilization, unresponsive UI. Modern Linux includes the OOM killer to terminate a process before total thrashing collapses the machine.

## Modern Reality: Swap Is Less Used

With cheap RAM (gigabytes per dollar), most workloads avoid swapping. Swap is now mostly:

- A safety net for rare memory spikes.
- A way to evict cold anonymous pages, freeing RAM for the page cache.
- Compressed in memory (Linux **zswap** / **zram**) to avoid the disk altogether.

Many cloud VM images ship with **no swap** because swapping is so costly that it's better to fail fast (OOM kill) than degrade.

## Related Notes

- [[Page Fault]] — the trap that triggers a swap-in.
- [[Page-Fault Handler]] — the OS code that does the I/O.
- [[Page Replacement]] — what to evict to swap out.
- [[Demand Paging]] — the policy that lazily brings pages in.
- [[Memory Hierarchy]] — swap is the disk tier under RAM.
- [[Page Table Entry]] — its PFN field doubles as a swap-block pointer when the page isn't present.
- [[Ch 21 — Beyond Physical Memory - Mechanisms]].
