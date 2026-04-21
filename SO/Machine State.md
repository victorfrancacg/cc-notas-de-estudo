---
title: Machine State
tags:
  - concept
  - ostep
  - process
aliases:
  - Process Machine State
type: concept
introduced_in: Ch 4
---

# Machine State

> [!abstract] Definition
> **Machine state** (of a process) is the complete set of things the process can **read or update** while executing — enough, in principle, to capture "everything about the process at this instant".

If you snapshot the machine state of a process, pause it, and later restore that snapshot, the process should resume as if nothing happened. That's the job of a [[Context Switch]].

## The Three Buckets

### 1. Memory — [[Address Space]]

Everything the process can load/store:

- **Code** — the instructions themselves.
- **Static data** — globals, constants.
- **[[Heap (Runtime)|Heap]]** — dynamically allocated memory (`malloc`).
- **[[Stack (Runtime)|Stack]]** — function frames, locals, return addresses.

### 2. Registers

- **General-purpose registers** — for arithmetic, arguments, temporaries.
- **[[Program Counter]]** — address of the next instruction to execute.
- **Stack pointer (SP)** — top of the call stack.
- **Frame pointer (FP / BP)** — base of the current function's frame.
- **Flags / status register** — condition codes from the last ALU op.

### 3. I/O Information

- **Open files** — the file descriptor table.
- **Current working directory**.
- **Signal handlers**, **credentials**, etc. (more advanced).

## Why This Carving Matters

When a process is **not running**, the OS must preserve exactly these things in its [[Process Control Block]]. When it resumes, the OS restores them, and the process never notices it was paused.

> [!tip] Symmetry with hardware
> The hardware's notion of "state" is mostly registers. The OS extends this to include memory and I/O info because those affect observable process behavior even if the CPU never "looks at them" directly.

## What's *Not* Machine State

- **Kernel-internal bookkeeping** (scheduling queues, timers, etc.) — it's about the process, but owned and managed by the OS.
- **Hardware cache contents** — conceptually transparent; performance-visible but not correctness-visible.

## Related Notes

- [[Process]], [[Process Control Block]], [[Address Space]].
- [[Context Switch]] — saves and restores machine state.
- [[Program Counter]], [[Register Context]].
