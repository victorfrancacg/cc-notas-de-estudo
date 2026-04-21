---
title: Zombie Process
tags:
  - concept
  - ostep
  - process
aliases:
  - Zombie
  - Defunct Process
type: concept
introduced_in: Ch 4
detailed_in: Ch 5
---

# Zombie Process

> [!abstract] Definition
> A **zombie** is a process that has **exited** but whose exit status has not yet been collected by its parent via [[wait()]]. Its [[Process Control Block]] lingers in the kernel just long enough for the parent to read the return code.

## Why It Exists

The kernel has to store *something* until the parent asks "how did my child finish?". Throwing away a child's exit status the instant it exits would destroy information the parent might need.

So on UNIX: a terminated process goes into the **zombie state**. When the parent calls [[wait()]], the kernel:

1. Returns the child's exit code to the parent.
2. Reaps the child — frees its PCB and slot.

## The Good Kind

A short-lived zombie is normal. Every well-written program that spawns children eventually `wait`s on them. The zombie lives for milliseconds.

## The Bad Kind

A **persistent** zombie is a bug: the parent isn't calling `wait()` and the process slot is leaking.

- Visible in `ps` as `<defunct>` or state `Z`.
- In long-running servers, zombies accumulate and can exhaust the kernel's process table.

## What if the Parent Dies First?

The zombie becomes an [[Orphan Process|orphan]] first, then gets re-parented to `init` (PID 1) — which calls `wait()` periodically in a loop. This keeps zombies from outliving their parents forever.

## Why "Zombie"?

OSTEP's footnote:

> Yes, the zombie state. Just like real zombies, these zombies are relatively easy to kill. However, different techniques are usually recommended.

## Related Notes

- [[Process]], [[Process States]], [[Process Control Block]].
- [[wait()]] — how a parent reaps a zombie.
- [[Orphan Process]] — cousin condition.
- [[Ch 5 — Interlude - Process API]] — full life-cycle handling.
