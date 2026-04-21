---
title: Von Neumann Model
tags:
  - concept
  - architecture
  - foundations
type: concept
introduced_in: Ch 2
---

# Von Neumann Model

> [!abstract] Definition
> The **Von Neumann model** of computing describes a machine that executes instructions in a simple loop: **fetch → decode → execute**, with instructions and data sharing a single memory.

## The Cycle

1. **Fetch** the next instruction from memory (at the address in the program counter).
2. **Decode** the instruction — figure out what it wants (add? load? branch?).
3. **Execute** it — perform the actual operation (update registers, touch memory, jump).
4. Advance the program counter; repeat.

## Why It Matters for OS

Virtually everything the OS does sits on top of this simple model. The OS is *itself* a program being fetched-decoded-executed. Its power comes from hardware support that lets kernel-mode instructions do things user-mode ones can't ([[Trap]]s, privileged instructions, interrupts).

## Modern Reality

> [!warning] Real CPUs cheat massively
> Modern processors execute many instructions at once (pipelining, superscalar), out of order, speculatively, across cores — all while maintaining the **appearance** of sequential fetch-decode-execute. OS-level reasoning still uses the pure Von Neumann model because the cheats are mostly invisible.

## Named for…

John von Neumann, one of the early pioneers of computing. Also — per OSTEP's footnote — he did early work on game theory, atomic bombs, and "played in the NBA for six years" *(one of these isn't true)*.

## Related Notes

- [[Operating System]].
- [[System Call]] — how OS code gets invoked within the Von Neumann loop.
- [[Interrupt]] — how the OS interrupts the loop to regain control.
