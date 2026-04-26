---
title: Page Frame
tags:
  - concept
  - ostep
  - memory
  - paging
aliases:
  - Frame
  - Physical Frame
  - PFN
type: concept
introduced_in: Ch 18
---

# Page Frame

> [!abstract] Definition
> A **page frame** is a fixed-size slot of physical memory used to hold one [[Page]]. Frames are the unit of physical-memory allocation under [[Paging]]; each is identified by its **Physical Frame Number (PFN)** = (frame address) / (page size).

## Frames vs Pages

| Page | Frame |
|---|---|
| Lives in [[Address Space]] | Lives in physical RAM |
| Per-process | Global |
| Indexed by VPN | Indexed by PFN |
| Many → One frame | One → potentially many pages (sharing) |

The OS's job is to maintain the mapping. Hardware enforces it via the [[Page Table]] + [[TLB]].

## PFN

For a 4 KB page size and 32-bit physical addresses:

- 32 bits = 20 bits PFN + 12 bits offset.
- 2²⁰ frames = 1,048,576 frames = 4 GB physical memory.

A [[Page Table Entry|page table entry]] stores the PFN; concatenating it with the offset produces the physical address.

## Frame Allocation

The OS keeps a **free-frame list**. On a [[Page Fault|page fault]] (or eager allocation), it pops a free frame and assigns it to the faulting page.

### Frame management complexity

Modern kernels track far more than just "free / used":

- **Active / inactive lists** — for [[Page Replacement|replacement policy]] decisions.
- **Anonymous vs file-backed** — different replacement semantics.
- **NUMA node** — frames near each CPU socket.
- **Migration / compaction** — to make huge-page contiguous regions available.

Linux uses the [[Buddy Allocator|buddy allocator]] for the free-frame list, which keeps coalesced regions large enough to satisfy huge-page requests.

## Sharing

The same physical frame can back **multiple virtual pages** in different processes:

- **Shared libraries** — one physical copy of `libc.so`, mapped into every process.
- **Forked processes (COW)** — parent and child share frames until either writes; on write, the kernel allocates a new frame and copies (copy-on-write, see [[fork()]]).
- **Shared memory IPC** — `shmget`, `mmap(MAP_SHARED)`.

## Related Notes

- [[Page]] — the virtual-side counterpart.
- [[Page Table]] — maps VPN → PFN.
- [[Page Table Entry]] — holds the PFN plus flags.
- [[Paging]] — the umbrella scheme.
- [[Physical Address]] — frame address + offset.
- [[Free-Space Management]] — the free-frame list.
- [[Ch 18 — Paging - Introduction]].
