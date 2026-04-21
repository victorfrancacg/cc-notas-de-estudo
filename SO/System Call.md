---
title: System Call
tags:
  - concept
  - ostep
  - mechanism
aliases:
  - Syscall
  - System Calls
type: concept
introduced_in: Ch 2
detailed_in: Ch 6
---

# System Call

> [!abstract] Definition
> A **system call** is a controlled entry point from user code into the kernel. Unlike a regular procedure call, it **raises the hardware privilege level** so the OS can do things user code cannot (touch devices, change memory mappings, create processes).

## The Key Difference from a Library Call

| | Library / procedure call | System call |
|---|---|---|
| Destination | Same program | [[Kernel]] |
| Privilege change | No | Yes: [[User Mode vs Kernel Mode\|user → kernel mode]] |
| How invoked | Direct jump | Special hardware instruction (a [[Trap]]) |
| How returned | `ret` | `return-from-trap` (also lowers privilege back) |

## Why It Exists

Without system calls, either:

- Every app would run with full hardware privileges (no [[Isolation (OS)]], catastrophic).
- Or apps couldn't talk to hardware at all (useless).

The trap instruction threads the needle: *give up control to the kernel at a specific, predefined point*, where the kernel can decide whether the request is allowed.

## Canonical Flow

1. User program calls a wrapper (e.g., `open()` in libc).
2. Wrapper puts arguments in registers and issues a **trap** instruction.
3. Hardware elevates privilege to [[Kernel Mode]] and jumps to a predefined [[Trap Handler]] (from the [[Trap Table]]).
4. Kernel validates args, performs the operation, places result in a register.
5. Kernel executes `return-from-trap`, lowering privilege back to user mode.
6. User code resumes with the result.

> [!info] Deeper in Ch 6
> [[Ch 6 — Mechanism - Limited Direct Execution]] dissects this flow — trap table setup, protection checks, cost of traps.

## Historical Note

System calls were **invented by the Atlas computing system** (early 1960s). Before that, OSes were just libraries — no privilege barrier between app and OS.

## Everyday Examples

- Process: `fork()`, `exec()`, `wait()`, `exit()`.
- Files: `open()`, `read()`, `write()`, `close()`.
- Memory: `mmap()`, `brk()`.
- Time / signals: `gettimeofday()`, `kill()`.

## Related Notes

- [[User Mode vs Kernel Mode]], [[Trap]], [[Trap Table]], [[Kernel]].
- [[Limited Direct Execution]] — the umbrella technique that relies on system calls.
- [[Process API]] — a family of system calls explored in Ch 5.
