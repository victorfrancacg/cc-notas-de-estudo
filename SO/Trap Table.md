---
title: Trap Table
tags:
  - concept
  - ostep
  - mechanism
aliases:
  - Interrupt Descriptor Table
  - IDT
  - Trap Vector
type: concept
introduced_in: Ch 6
---

# Trap Table

> [!abstract] Definition
> A **trap table** (x86: **Interrupt Descriptor Table**, IDT) is a kernel-owned array that maps trap / interrupt numbers to handler addresses. The hardware consults this table whenever a [[Trap]] fires, using the table entry to know which kernel code to jump to.

## Why It Exists

Without a table, either:

- User code would specify the destination address of a trap — catastrophic, since a wily process could jump past security checks into the middle of privileged code.
- All traps would go to one fixed address — clumsy, forcing software to demultiplex every trap manually.

The table gives the kernel a pre-registered, controlled set of entry points, one per kind of event.

## Typical Contents

| Entry | Reason |
|---|---|
| 0 | Divide-by-zero exception |
| 3 | Breakpoint |
| 6 | Invalid opcode |
| 13 | General protection fault |
| 14 | Page fault |
| 32+ | Hardware [[Interrupt|interrupts]] (timer, keyboard, disk, …) |
| 128 (linux i386) | `int 0x80` — the classic syscall trap |

(Exact numbers are architecture-specific.)

## Setting It Up — Privileged, One-Time

The kernel installs the trap table at **boot**, using a **privileged instruction** (`lidt` on x86). After boot, the table is effectively immutable from user mode — user code has no way to change it.

## Flow on a Trap

1. User code (or hardware) causes a trap with some number N.
2. Hardware saves relevant registers, switches to kernel mode.
3. Hardware reads entry N from the trap table.
4. Hardware jumps to the handler address at entry N.
5. Handler runs (as OS code), eventually issues return-from-trap.

## Related Notes

- [[Trap]]
- [[System Call]]
- [[Limited Direct Execution]]
- [[Interrupt]], [[Timer Interrupt]]
