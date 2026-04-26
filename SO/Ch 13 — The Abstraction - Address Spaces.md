---
title: Ch 13 — The Abstraction - Address Spaces
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
type: chapter
book: OSTEP
chapter: 13
pages: 131-140
---

# Ch 13 — The Abstraction: Address Spaces

> [!abstract] One-sentence summary
> The OS gives each [[Process]] the illusion of a large, private, contiguous **[[Address Space]]** — code at the bottom, [[Heap (Runtime)|heap]] growing up, [[Stack (Runtime)|stack]] growing down. Behind the scenes physical memory is shared and fragmented, but with hardware help (Ch 15+) the illusion is fast enough to use. The chapter sets the **goals** of any virtual-memory system: *transparency, efficiency, protection*.

## Crux

> [!example] The Crux: How To Virtualize Memory
> How can the OS build the abstraction of a private, potentially large [[Address Space]] for multiple running [[Process|processes]] (all sharing memory) on top of a single, physical memory?

## 13.1 Early Systems

> [!info] Single-program model
> Memory looked like:
> ```
>   0KB  ┌────────────────────────┐
>        │ Operating System       │
>        │ (code, data, etc.)     │
>  64KB  ├────────────────────────┤
>        │ Current Program        │
>        │ (code, data, etc.)     │
>   max  └────────────────────────┘
> ```

- The OS was just a **library of routines** sitting in memory (e.g., at physical address 0).
- Exactly **one** running program — at some other physical address — used the rest of RAM.
- No abstraction needed. Programmers expected nothing fancy. Life was easy for OS authors.

## 13.2 Multiprogramming and Time Sharing

Two pressures broke the early model:

1. **[[Multiprogramming]]** — keep several processes "ready", switch when one does I/O. Increases CPU [[OS Design Goals|utilization]] (machines cost millions; idle CPU is waste).
2. **[[Time Sharing]]** — multiple users at terminals expecting **[[Cooperative vs Preemptive Scheduling|interactivity]]**.

### First (bad) idea: full-memory swap

Run process A for a slice → **save all of physical memory to disk** → load process B → run.

> [!warning] Why this fails
> Saving an entire memory image to disk on every [[Context Switch|context switch]] is *brutally slow*. Memory grew faster than disk bandwidth.

### Better idea: leave processes resident

Carve physical memory so multiple processes coexist; only swap CPU state on context switch.

```
   0KB  ┌─────────────────────────┐
        │ Operating System        │
  64KB  ├─────────────────────────┤
        │ (free)                  │
 128KB  ├─────────────────────────┤
        │ Process C               │
 192KB  ├─────────────────────────┤
        │ Process B               │
 256KB  ├─────────────────────────┤
        │ (free)                  │
 320KB  ├─────────────────────────┤
        │ Process A               │
 384KB  ├─────────────────────────┤
        │ (free)                  │
 512KB  └─────────────────────────┘
```

This is what creates the **need for protection**: with multiple processes resident, A must not be able to read or write B's memory — see [[Isolation (OS)]].

## 13.3 The Address Space — The Abstraction

> [!abstract] Definition
> A process's **[[Address Space]]** is the OS's clean abstraction of memory for that process: a large, private, contiguous range of bytes containing **code**, **heap**, **stack**, and statics. Physical placement is the OS's problem; the program just sees the abstraction.

### Three canonical regions

| Region | Direction | Purpose |
|---|---|---|
| **[[Code Segment\|Code]]** | static | program instructions; read-only; doesn't grow |
| **[[Heap (Runtime)\|Heap]]** | grows up ↓ | dynamic allocation (`malloc`, `new`) |
| **[[Stack (Runtime)\|Stack]]** | grows down ↑ | call frames, locals, return addresses |

```
   0KB  ┌──────────────────┐
        │ Program Code     │  static
   1KB  ├──────────────────┤
        │ Heap             │  grows ↓
   2KB  │     ↓            │
        │  (free)          │
        │     ↑            │
  15KB  │ Stack            │  grows ↑
  16KB  └──────────────────┘
```

> [!tip] Why heap and stack grow toward each other
> By placing them at opposite ends of the address space, **either** can grow without colliding with the other until they meet in the middle. The convention isn't sacred — multi-threaded programs lay out *one stack per thread* and break it.

### The illusion

When process A reads "address 0", the OS + hardware redirect the access to whatever physical address actually holds A's code (e.g., 320KB). Process B reads "address 0" and gets a *different* physical location. To each process, its address space starts at 0 and runs up to its declared maximum — **a private, contiguous illusion**.

## 13.4 Goals

The OS's job in this part of the book is to **virtualize memory with style**. Three explicit goals:

> [!info] Transparency
> The program must be **unaware** that memory is virtualized. It should behave as if it had its own private physical memory. Every access "just works".
>
> *Note:* "transparent" here means *invisible/undetectable* — the opposite of its everyday "open and visible" meaning.

> [!info] Efficiency
> Both **time** (programs shouldn't slow down because of translation) and **space** (the OS shouldn't burn huge amounts of memory on bookkeeping). Hardware features ([[TLB]]s, page-table walkers) exist precisely for this goal.

> [!info] Protection
> Processes must be **[[Isolation (OS)|isolated]]** from each other and from the OS. Misbehavior in process A cannot read or corrupt process B's address space, nor the kernel's. Modern systems get this *for free* once the address-translation hardware refuses out-of-bounds accesses.

## 13.5 Summary

> [!example] Crux answered
> **How does the OS virtualize memory?** By giving each process an address space — a large, private, contiguous illusion — and using hardware-assisted translation to map every [[Virtual Address|virtual address]] the process generates onto an actual physical location. The rest of [[MOC - Memory Virtualization|Part II]] is mechanism (base/bounds, segmentation, paging, TLBs) and policy (allocation, replacement) for making this work efficiently and safely.

## Aside: A Common Source of Confusion

> [!warning] "What's the address of `x`?" is ambiguous
> When a program prints the address of a variable via `printf("%p", &x)`, the number printed is a **[[Virtual Address]]** in the program's own address space — not a physical RAM location. Two runs of the same program will likely print *the same* virtual address while occupying *different* physical memory.

## Related Notes

- [[Address Space]] — the abstraction itself, expanded.
- [[Virtual Address]] vs Physical Address.
- [[Memory Virtualization]] — the umbrella concept.
- [[Code Segment]], [[Heap (Runtime)]], [[Stack (Runtime)]] — the layout pieces.
- [[Isolation (OS)]] — the protection goal in concrete terms.
- [[Process]] — every process has exactly one address space.
- [[Multiprogramming]], [[Time Sharing]] — the historical pressures.
- Next: [[Ch 14 — Interlude - Memory API]] — how user code asks for and frees memory.
- Index: [[MOC - Memory Virtualization]].
