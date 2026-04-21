---
title: Isolation (OS)
tags:
  - concept
  - ostep
  - design-principle
  - foundations
aliases:
  - Process Isolation
  - OS Isolation
type: concept
introduced_in: Ch 2
---

# Isolation (OS)

> [!abstract] Definition
> **Isolation** is the OS guarantee that one process's misbehavior — whether buggy or malicious — **cannot damage** other processes or the OS itself. It's a cornerstone of every modern OS and the reason the word "protection" appears in [[OS Design Goals]].

## Why It's Fundamental

Without isolation:

- A buggy program could scribble into another program's memory → crashes everywhere.
- A malicious program could read passwords, keys, personal data of others.
- A process in an infinite loop could lock out the OS entirely.

Essentially, you'd be back to the pre-1960s mainframe world where *one* running program controlled the whole machine.

## The Mechanisms That Provide Isolation

Isolation is never a single feature — it's enforced by the combination of:

| Mechanism | What it isolates |
|---|---|
| [[User Mode vs Kernel Mode]] | User code cannot execute privileged instructions. |
| [[Address Space]] | Each process has its own memory view; can't see others' memory. |
| [[Paging]] + MMU (Part I memory) | Hardware-enforced memory protection. |
| [[System Call]] + [[Trap Table]] | Controlled gateway to privileged operations. |
| [[Timer Interrupt]] | OS can always regain the CPU. |
| File permissions, UIDs/GIDs | Access control for persistent data. |

## The "Reboot Safety" Argument

> [!tip] A classic Lampson observation
> If the OS is well-isolated, a buggy application just **dies** and the system keeps running. If isolation is weak, one application's bug reboots the machine.
>
> Arguably the whole point of a modern OS is: *contain failures to the failing component*.

## Isolation vs Sharing

Processes need to *share* some things (files, pipes, IPC, shared memory). Isolation isn't "nothing shared" — it's "nothing shared **without permission**":

- Files are shared through explicit `open()` calls.
- Shared memory exists via explicit `mmap(MAP_SHARED)` / `shmget()`.
- IPC channels are explicitly created.

The default is isolation; sharing requires effort.

## Isolation at Higher Levels

The same principle repeats up the stack:

- **Containers** (Docker, LXC) — process-group isolation.
- **Virtual machines** — whole-OS isolation.
- **Browser sandboxes** — tab-level isolation.
- **Language runtimes** (e.g., WebAssembly) — function-level isolation.

Each layer adds a new isolation boundary for its specific threat model.

## Related Notes

- [[User Mode vs Kernel Mode]], [[Address Space]], [[System Call]].
- [[Principle of Least Privilege]] — sibling design principle.
- [[OS Design Goals]].
- [[Operating System]].
- [[Memory Protection]].
