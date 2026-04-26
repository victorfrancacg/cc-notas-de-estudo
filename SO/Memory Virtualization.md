---
title: Memory Virtualization
tags:
  - concept
  - ostep
  - memory
  - virtualization
  - foundations
aliases:
  - Virtual Memory
  - VM (Memory)
type: concept
introduced_in: Ch 12
detailed_in: Ch 13
---

# Memory Virtualization

> [!abstract] Definition
> **Memory virtualization** is the OS's act of giving each [[Process]] the illusion of a large, private [[Address Space]] on top of a single shared physical memory. Every [[Virtual Address]] the program generates is translated — at hardware speed — to a physical location. The umbrella concept of the [[MOC - Memory Virtualization|memory half of Part I]] in OSTEP (Ch 12–24).

## The Three Goals

> [!info] Transparency
> The program is **unaware** that memory is virtualized. Behaviour matches "I have my own private memory."

> [!info] Efficiency
> Translation must be **fast** (hardware [[MMU]] + [[TLB]]) and **space-efficient** ([[Multi-Level Page Table|multi-level]] / [[Inverted Page Table|inverted]] page tables when memory is scarce).

> [!info] Protection
> Processes are **[[Isolation (OS)|isolated]]** — no process can read or write another's memory or the kernel's.

(These mirror the [[OS Design Goals]] but specialized to memory.)

## Why Bother

- **[[Multiprogramming]]** keeps the CPU busy by letting many processes coexist in memory simultaneously — but only if each can be safely placed *somewhere* without colliding.
- **[[Time Sharing]]** users expect interactive responsiveness; can't afford to swap the entire RAM image to disk on every context switch.
- **Bug containment** — a wild pointer in one program shouldn't corrupt another. The OS turns out-of-bounds accesses into fatal traps via the translation hardware.
- **Programmer simplicity** — code that runs as PID 1234 today and PID 6789 tomorrow uses the same virtual addresses and Just Works.

## How It's Built — Roadmap

```
       SIMPLEST                                     MOST CAPABLE
  ┌──────────────────┐                          ┌──────────────────┐
  │  base + bounds   │  → segments → paging →   │  multi-level PT  │
  │  (Ch 15)         │   (Ch 16)    (Ch 18)     │  (Ch 20) + TLB   │
  └──────────────────┘                          └──────────────────┘
```

Each stage relaxes a limitation:

- **Base/bounds**: only one contiguous slot per process — no sparse address spaces.
- **Segmentation**: multiple slots, but external fragmentation hurts.
- **Paging**: fixed-size pages eliminate fragmentation; page tables become huge.
- **TLBs**: cache recent translations so page lookups don't dominate runtime.
- **Multi-level / inverted page tables**: shrink the page-table footprint itself.

## Key Ingredients

- **Hardware**: the [[MMU]] (translates), the [[TLB]] (caches), trap-on-fault circuits.
- **OS data structures**: per-process [[Page Table|page tables]], the [[Free-Space Management|free-frame list]], swap maps.
- **OS policies**: page allocation, page replacement (Ch 22), where to put new processes.

## Cross-Cutting Concerns

> [!warning] Translation is on the *hot path*
> Every load/store goes through it. A 10× slowdown in translation is a 10× slowdown in the program. Hence the obsession with [[TLB]] hit rates and prefetched walks.

> [!warning] Memory protection ≠ memory encryption
> Virtualization protects from *other processes*. It does not protect from a privileged kernel, from cold-boot attacks, or from snooping over the memory bus.

## Related Notes

- [[Virtualization]] — the broader concept (CPU side is [[Process]], memory side is [[Address Space]]).
- [[Address Space]] — the abstraction memory virtualization delivers.
- [[Virtual Address]] — what user programs generate.
- [[MMU]], [[TLB]], [[Page Table]] — the mechanism stack.
- [[Isolation (OS)]] — the protection goal made concrete.
- [[Three Easy Pieces]] — virtualization is one of them.
- [[MOC - Memory Virtualization]] — the part-level index.
