---
title: Operating System
tags:
  - concept
  - ostep
  - foundations
aliases:
  - OS
  - Virtual Machine (OS sense)
type: concept
introduced_in: Ch 2
---

# Operating System

> [!abstract] Definition
> An **Operating System (OS)** is the body of software that makes it *easy to use* a computer by (a) **virtualizing** physical resources, (b) managing **concurrency**, and (c) providing **persistence** — while enforcing protection and fair sharing among applications.

## The Three Hats an OS Wears

1. **[[Virtualization|Virtual machine]]** — it presents each [[Process]] with the illusion of a dedicated CPU and dedicated memory. See [[Virtualization]].
2. **Standard library** — it exports a few hundred [[System Call|system calls]] (`fork`, `open`, `mmap`, etc.) that applications use.
3. **Resource manager** — it decides *who* gets *which* CPU, *how much* memory, *when* to run, and *where* data lives on disk. It arbitrates, guided by [[OS Design Goals|design goals]] (fairness, throughput, latency).

## What's Underneath

At the bottom sits the [[Von Neumann Model]]: a processor fetching, decoding, executing instructions. The OS is a program too — its only "privilege" is that the hardware lets it do things user code can't (see [[User Mode vs Kernel Mode]]).

## What an OS Does *Not* Do (in OSTEP's framing)

- Deep networking code (belongs to a networking course).
- Graphics / GPU stacks.
- Security beyond process isolation.

## Brief History (see also [[Ch 2 — Introduction to Operating Systems]])

| Era | Shift |
|---|---|
| Libraries era | OS = shared routines; [[Batch Processing]] with a human operator deciding job order. |
| Protection era | [[System Call]] invented (Atlas); hardware privilege levels; [[Trap]] instruction. |
| Multiprogramming / UNIX era | Memory protection, concurrency awareness, C language, open-source culture. |
| PC era | Regression (DOS, classic Mac OS lacked protection). |
| Modern era | Convergence — macOS (UNIX core), Windows NT, Linux (Torvalds), Android (Linux kernel). |

## Design Principles at the Core

- [[Mechanism vs Policy]] — keep them separate.
- [[Isolation (OS)]] — a bad/compromised app must not sink the whole system.
- [[Principle of Least Privilege]] — give each component the minimum power it needs.
- Abstraction — pretty much always, choose a clean abstraction over a leaky one.

## Related Notes

- [[Virtualization]], [[Concurrency]], [[Persistence]] — the three pieces.
- [[OS Design Goals]] — what we're optimizing for.
- [[System Call]] — the OS's public API.
- [[Kernel]] — the protected core of the OS.
- [[MOC - OSTEP]] — index of the book.
