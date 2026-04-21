---
title: Virtualization
tags:
  - concept
  - ostep
  - foundations
  - virtualization
aliases:
  - Virtualize
  - Virtualizing
type: concept
introduced_in: Ch 2
---

# Virtualization

> [!abstract] Definition
> **Virtualization** is the technique by which the OS takes a **physical** resource (CPU, memory, disk) and transforms it into a more **general**, **powerful**, and **easy-to-use virtual** form of that same resource.

It's one of the [[Three Easy Pieces|three big things an OS does]] and the focus of Part I of [[MOC - OSTEP|OSTEP]].

## The Core Idea

The OS provides an **illusion**: each running process acts as if it owns the machine. It has "its own" CPU and "its own" memory — even though many processes share the real hardware.

> [!note] Why "virtualization" is the right word
> The OS is presenting a *virtual* version of a real resource — just like a virtual machine presents virtual hardware to a guest OS. Historically, OSes were literally called **virtual machines**.

## Two Flavors in Part I

### Virtualizing the CPU — [[Time Sharing]]

- Physical: one (or a few) CPU cores.
- Virtual: each [[Process]] runs as if it has its own CPU.
- Trick: rapidly switch between processes ([[Context Switch]]). Each runs for a small slice; the illusion emerges from the speed.
- Requires: [[Limited Direct Execution]] (safety), a [[Scheduler]] (policy), and [[Timer Interrupt|timer interrupts]] (control).

### Virtualizing Memory — [[Address Space]]

- Physical: a single big array of bytes (DRAM).
- Virtual: each process gets its *own* private [[Address Space]], starting at address 0.
- Trick: the OS + hardware translate every memory access from a *virtual* address to a *physical* address on the fly.
- Two processes can each store data at `0x200000` without conflict — the translations diverge.

## Why Virtualize?

1. **Ease of use** — programmers don't coordinate who uses which CPU or memory region.
2. **Protection** — bad behavior in one process can't scribble over another's data. See [[Isolation (OS)]].
3. **Utilization** — overlap CPU with I/O by switching ([[Multiprogramming]]).
4. **Abstraction** — hardware differences hidden behind a stable API.

## The Interface: System Calls

Users tell the OS what to do through a standard library of **hundreds of [[System Call|system calls]]** (e.g., `fork()`, `open()`, `mmap()`). Virtualization is useless without an API through which to request virtualized resources.

## Virtualization ≠ Persistence

> [!warning] Not everything is virtualized
> The OS does *not* privately virtualize the disk the way it virtualizes memory. Files are meant to be **shared** across processes and users — the editor writes `foo.c`, the compiler reads it. See [[Persistence]], [[File System]].

## Related Notes

- [[Operating System]] — the software that performs virtualization.
- [[Mechanism vs Policy]] — virtualization is built from both.
- [[Time Sharing]], [[Space Sharing]] — the two fundamental sharing strategies.
- [[Process]] — what gets virtualized on the CPU side.
- [[Address Space]] — what gets virtualized on the memory side.
- [[Concurrency]] — the problem virtualization creates (many things happening at once).
- [[MOC - CPU Virtualization]] — the CPU-side deep dive.
