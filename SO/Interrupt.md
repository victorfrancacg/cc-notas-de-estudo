---
title: Interrupt
tags:
  - concept
  - ostep
  - mechanism
  - hardware
aliases:
  - Hardware Interrupt
type: concept
introduced_in: Ch 6
---

# Interrupt

> [!abstract] Definition
> An **interrupt** is an **asynchronous** hardware-initiated signal that forces the CPU to stop whatever it's doing, save its state, and jump to a pre-registered kernel handler. It's the mechanism by which the OS learns about external events — clocks, disks, keyboards, network cards.

## Interrupt vs Exception vs System Call

All three funnel through the same [[Trap]] machinery, but differ in origin:

| Kind | Caused by | Synchronous? |
|---|---|---|
| **Interrupt** | External hardware (timer, device, NIC) | **Asynchronous** |
| **Exception** | Current instruction (div by zero, page fault) | Synchronous |
| **System call** | User code deliberately | Synchronous |

From the hardware's perspective, all three save state, raise privilege, and consult the [[Trap Table]]. The OS handler tells them apart.

## Why Interrupts Exist

Polling would be wasteful: the CPU would repeatedly ask "is the disk done yet?" in a loop. With interrupts, the CPU runs useful work until the device itself says "I'm done!" — at which point the device raises an interrupt line, the CPU vectors to the handler, and the OS deals with the event.

## Interrupt Lifecycle

1. Device (or timer) asserts an interrupt line.
2. CPU finishes its current instruction.
3. CPU saves state (PC, flags, some registers) onto the kernel stack.
4. CPU switches to kernel mode.
5. CPU jumps to the handler listed in the [[Trap Table]].
6. Handler processes the event quickly (e.g., reads a buffer from a device register, wakes up a blocked process).
7. Handler issues return-from-trap.

## Kinds of Interrupts

| Source | Example event |
|---|---|
| [[Timer Interrupt|Timer]] | Scheduling tick |
| Disk | I/O complete |
| NIC | Packet arrived |
| Keyboard | Key pressed |
| IPI (Inter-Processor Interrupt) | One CPU pokes another |

## Important Properties

- **Asynchronous** — can happen mid-instruction, mid-syscall, mid-critical-section.
- **Maskable** — the kernel can temporarily disable most interrupts during critical sections. `NMI` (non-maskable) is the exception.
- **Priority** — hardware usually supports multiple priority levels.

## Interrupts and Concurrency

> [!warning] Interrupts are the original concurrency
> Long before threads, OS programmers had to reason about "what if an interrupt fires right here?" This is why kernels are written so carefully around shared structures. See [[Concurrency]].

## Related Notes

- [[Timer Interrupt]] — the most important kind for scheduling.
- [[Trap]], [[Trap Table]] — the shared machinery.
- [[Device Driver]] — typical interrupt consumer.
- [[System Call]], exception — sibling events.
