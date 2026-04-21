---
title: Process
tags:
  - concept
  - ostep
  - process
  - foundations
aliases:
  - Process (OS)
  - Running Program
type: concept
introduced_in: Ch 4
---

# Process

> [!abstract] Definition
> A **process** is the OS's abstraction of a **running program**. A program on disk is lifeless bytes; the OS loads it, sets up its runtime, and begins executing it — and that live instance is a process.

## Program vs Process

| **Program** | **Process** |
|---|---|
| Static bytes on disk | Live instance, executing |
| No state beyond its file | Full [[Machine State]]: memory, registers, I/O |
| One file | One *or many* processes (run the same program twice → two processes) |
| Inert | Has a PID, a parent, a state, a lifecycle |

## What a Process Consists Of — [[Machine State]]

At any instant, a process is described by:

### 1. Memory ([[Address Space]])

Its private view of memory — code, static data, [[Heap (Runtime)|heap]], [[Stack (Runtime)|stack]].

### 2. Registers

General-purpose registers plus special ones:

- **[[Program Counter]]** (PC, a.k.a. instruction pointer / IP) — address of the next instruction.
- **Stack pointer (SP)** — top of the run-time stack.
- **Frame pointer** — base of the current function's stack frame.

### 3. I/O information

Open files (in UNIX, the file descriptor table), current working directory, etc.

## Lifecycle States

Simplified three-state model:

- **Running** — executing on a CPU.
- **Ready** — runnable, waiting for the [[Scheduler]] to pick it.
- **Blocked** — waiting for an event (I/O, signal, child exit).

Plus edge states:

- **Initial** / **Embryo** — being created.
- **[[Zombie Process|Zombie]]** — has exited, parent hasn't `wait()`ed yet.

See [[Process States]] for the full diagram.

## The OS's Per-Process Bookkeeping

Every process has a [[Process Control Block]] (PCB), stored in a [[Process List]]. The PCB holds everything the OS needs to pause and resume the process.

## API

- **Create**: [[fork()]], [[exec()]].
- **Wait / synchronize**: [[wait()]].
- **Destroy**: `kill()`, `exit()`.
- **Inspect**: `getpid()`, `ps`.
- Full treatment in [[Ch 5 — Interlude - Process API]].

## Why We Have Processes

> [!tip] Processes buy you two things for free
> 1. **Isolation** — one process can't trample another ([[Isolation (OS)]], enabled by [[Address Space|per-process address spaces]]).
> 2. **Concurrency illusion** — the OS can pause one process and run another transparently ([[Time Sharing]], [[Context Switch]]).

## Process vs Thread

Not yet introduced at this point in OSTEP, but worth flagging:

- A process is "heavy" — it has its own address space.
- A [[Thread]] is "light" — it runs *inside* a process, sharing that process's address space with sibling threads. Threads are Ch 26+.

## Related Notes

- [[Machine State]] — what defines a process's state.
- [[Address Space]] — the memory half.
- [[Process Control Block]] — the kernel-side struct.
- [[Process States]] — the lifecycle.
- [[Context Switch]] — how the CPU moves from one process to another.
- [[Scheduler]] — who decides which process runs.
- [[fork()]], [[exec()]], [[wait()]] — the UNIX process API.
