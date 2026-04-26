---
title: Page Fault
tags:
  - concept
  - ostep
  - memory
  - paging
  - trap
aliases:
  - Page Miss
  - Major Fault
  - Minor Fault
type: concept
introduced_in: Ch 21
---

# Page Fault

> [!abstract] Definition
> A **page fault** is the trap raised by the [[MMU]] when a memory reference touches a [[Page|virtual page]] whose [[Page Table Entry|PTE]] has `Present = 0` — i.e., the page is mapped but not currently in physical memory. Hardware delivers control to the OS's [[Page-Fault Handler|page-fault handler]], which brings the page in from [[Swap Space|swap]] (or a file) and retries the instruction.

## The Distinction: Mapped vs Unmapped

| | Mapped (PTE.Valid=1) | Unmapped (PTE.Valid=0 or no PTE) |
|---|---|---|
| In RAM (Present=1) | normal access | impossible state |
| Not in RAM (Present=0) | **page fault** — OS swaps in | **[[Segmentation Fault\|segfault]]** — kill process |

Both are "the hardware traps to the OS"; the OS distinguishes by inspecting the PTE.

> [!warning] Casual usage is sloppy
> "Page fault" sometimes refers to *any* trap from the page-translation hardware, including illegal accesses. Strictly: page-fault = page-not-present (recoverable). Bad-access = protection or segmentation fault (typically fatal).

## Major vs Minor Page Faults

Linux/POSIX distinguish two kinds of recoverable faults:

| Major fault | Minor fault |
|---|---|
| Requires disk I/O — page has to be read from swap or file | No I/O — page is already in memory but not yet mapped to this process |
| Slow — milliseconds | Fast — microseconds |
| Counted in `getrusage()` as `ru_majflt` | Counted as `ru_minflt` |

**Minor faults** are common in modern systems:
- **Demand-zero pages** — anonymous heap pages allocated but not yet touched. First write triggers a minor fault; the kernel allocates a zeroed frame.
- **Shared library mappings** — process maps `libc.so`; first access is a minor fault that wires up to an existing in-memory page.
- **Copy-on-write after fork** — first write to a shared page causes a minor fault; kernel makes a private copy.

## Servicing Path

```
load *p   →  TLB miss  →  PTE.Present=0  →  page fault
                                                  ↓
                                        page-fault handler:
                                          find free PFN (or evict)
                                          (if backed) read from swap/file
                                          update PTE: Present=1, PFN
                                          mark process Ready
                                                  ↓
                                        retry instruction → success
```

While I/O is in flight, the process is **Blocked**; other processes run.

## Performance Impact

Page faults dominate VM-system performance because disk is so much slower than RAM:

| | Approximate cost |
|---|---|
| Cache hit | ~1 ns |
| RAM access | ~100 ns |
| Minor page fault | ~10 µs |
| Major page fault (NVMe) | ~100 µs |
| Major page fault (HDD) | ~10 ms |

A program with 1% major faults can run **10⁴× slower** than the same program with 0% faults.

## Related Notes

- [[Page-Fault Handler]] — the OS code that runs.
- [[Swap Space]] — where most major-fault content comes from.
- [[Demand Paging]] — relies on minor faults for first-touch allocation.
- [[Page Table Entry]] — present bit drives the fault.
- [[Page Replacement]] — Ch 22, what victim to evict.
- [[Segmentation Fault]] — the *unrecoverable* sibling.
- [[TLB]] — the cache that hides successful translations.
- [[Ch 21 — Beyond Physical Memory - Mechanisms]].
