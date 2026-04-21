---
title: Trap
tags:
  - concept
  - ostep
  - mechanism
aliases:
  - Trap Instruction
  - Software Interrupt
type: concept
introduced_in: Ch 6
---

# Trap

> [!abstract] Definition
> A **trap** is a hardware mechanism that synchronously transfers control from user code into the kernel, **while simultaneously raising the CPU's privilege level** from user to kernel mode. It's the one-way door that makes [[System Call|system calls]] and other privileged operations possible without compromising safety.

## What Hardware Does on a Trap

1. **Save registers** (including the [[Program Counter|PC]], flags, and a few more) onto the per-process **kernel stack**.
2. **Elevate privilege** from user to kernel mode.
3. **Jump to a pre-registered handler** — an entry in the [[Trap Table]].

The reverse ("return-from-trap") pops those registers back and lowers privilege.

## What Triggers a Trap?

Three flavors, all funnel through the trap machinery:

| Kind | Example | Synchronous? |
|---|---|---|
| **System call** | `read()`, `write()` | Synchronous (caused by user code) |
| **Exception** | Divide-by-zero, invalid memory access | Synchronous (caused by user code) |
| **[[Interrupt]]** | [[Timer Interrupt]], disk I/O complete | **Asynchronous** (external) |

All three use the same hardware mechanism — the trap table entries distinguish them.

## The Assembly-Language Reality

When you call `open()` in C, you don't write a trap instruction yourself. The C library (libc) wraps it:

```asm
; hand-written libc stub for open()
mov  rax, SYS_open     ; syscall number
mov  rdi, path
mov  rsi, flags
syscall                ; the trap instruction on x86-64
ret
```

The `syscall` instruction is the trap. Return values come back in registers.

## Why Users Can't Pick the Destination

A naïve design might let user code say "trap to address X". That would let user code jump anywhere in the kernel and bypass security checks. Instead:

- Handlers are registered in the **[[Trap Table]]** at boot (privileged op).
- The trap instruction only jumps to pre-registered entries.
- User code picks only the *system-call number* — the kernel decides what that number means.

## Trap Table

See [[Trap Table]] for details. In short: a kernel-owned table mapping trap numbers → handler addresses. Installed once at boot; immutable from user mode.

## Related Notes

- [[System Call]] — the user-visible abstraction a trap implements.
- [[Trap Table]]
- [[User Mode vs Kernel Mode]]
- [[Limited Direct Execution]]
- [[Interrupt]], [[Timer Interrupt]]
