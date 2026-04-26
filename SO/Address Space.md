---
title: Address Space
tags:
  - concept
  - ostep
  - memory
  - virtualization
aliases:
  - Virtual Address Space
  - Process Address Space
type: concept
introduced_in: Ch 2
detailed_in: Ch 13
---

# Address Space

> [!abstract] Definition
> A process's **address space** is the (virtual) view of memory it sees — a flat array of bytes, indexed from 0 up to some maximum, containing its code, stack, heap, and globals. The OS + hardware transparently map these virtual addresses onto physical memory locations.

## The Illusion

When a process reads address `0x200000`, it *thinks* it's reading "memory slot 0x200000". Actually:

- Each process has its **own** address space.
- Two processes can both "read `0x200000`" and get completely different values — because the OS maps their virtual `0x200000` to *different* physical addresses.

> [!example] The `./mem` demo (Ch 2.2)
> Two simultaneous runs of a program that prints the address returned by `malloc` both print `0x200000`, yet each keeps its own counter there and never sees the other's value. That's the illusion working.

## Why This Matters

- **[[Isolation (OS)|Isolation]]** — a buggy process can't scribble into another's memory. It can only touch its own address space.
- **Simplicity for programmers** — no coordination over "who owns which byte of DRAM".
- **Flexibility** — the OS is free to place pages wherever on physical RAM (or swap them out to disk).

## What's Inside an Address Space

A typical process address space layout:

```
high addresses  +-----------+
                |  stack    |  (grows down)
                |    ↓      |
                |           |
                |    ↑      |
                |  heap     |  (grows up)
                +-----------+
                |  bss/data |
                +-----------+
                |  code     |
low addresses   +-----------+
```

This is a **logical** layout — the physical placement can be arbitrary.

The three canonical regions ([[Ch 13 — The Abstraction - Address Spaces|Ch 13]]):

- **[[Code Segment]]** — static instructions, read-only. Doesn't grow.
- **[[Heap (Runtime)]]** — dynamic allocations (`malloc`, `new`). Grows toward higher addresses.
- **[[Stack (Runtime)]]** — call frames, locals, return addresses. Grows toward lower addresses.

## The Three Goals (Ch 13.4)

> [!info] Transparency, Efficiency, Protection
> 1. **Transparency** — the program is unaware memory is virtualized.
> 2. **Efficiency** — translation is fast and the OS uses minimal bookkeeping space.
> 3. **Protection** — processes are [[Isolation (OS)|isolated]] from each other and from the kernel.

## How the Illusion Is Built — Preview

Ch 13+ goes deep. In short:

- **Base-and-bounds** (Ch 15) — a simple shift.
- **Segmentation** (Ch 16) — separate base/bounds per region.
- **[[Paging]]** (Ch 18) — fixed-size chunks, a [[Page Table]] per process.
- **[[TLB]]** (Ch 19) — caches translations for speed.
- **[[Swap Space]]** (Ch 21) — address space bigger than RAM; overflow goes to disk.

## Related Notes

- [[Virtualization]] — address space *is* the memory-side instance of it.
- [[Process]] — every process has one address space.
- [[Paging]], [[Page Table]], [[TLB]] — the mechanisms that make it work.
- [[Virtual Memory]] — the umbrella term for the whole system.
