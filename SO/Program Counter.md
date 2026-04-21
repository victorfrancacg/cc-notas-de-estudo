---
title: Program Counter
tags:
  - concept
  - ostep
  - architecture
  - process
aliases:
  - PC
  - Instruction Pointer
  - IP
type: concept
introduced_in: Ch 4
---

# Program Counter

> [!abstract] Definition
> The **program counter (PC)**, also known as the **instruction pointer (IP)**, is the CPU register that holds the address of the **next instruction** to execute.

## Its Role in the [[Von Neumann Model]]

Every iteration of fetch→decode→execute starts by reading from `[PC]`. After a non-branching instruction, the hardware advances the PC to the next instruction. Branches, calls, returns, interrupts, and traps write to the PC directly.

## Why It's Special for the OS

The PC is the **most important** register saved during a [[Context Switch]]. Save it wrong, and when the process resumes it jumps to a bizarre address and crashes.

It's also what makes a [[Trap]] work: the trap instruction captures the faulting PC so the kernel knows *where* the process was when it was interrupted.

## Conventional Names

| Architecture | Name |
|---|---|
| x86 (32-bit) | `eip` (extended instruction pointer) |
| x86-64 | `rip` |
| ARM | `pc` |
| RISC-V | `pc` |
| MIPS | `pc` |

## Related Notes

- [[Von Neumann Model]]
- [[Context Switch]]
- [[Machine State]]
- [[Trap]]
