---
title: Ch 3 — A Dialogue on Virtualization
tags:
  - ostep
  - chapter
  - part/virtualization
  - dialogue
type: chapter
book: OSTEP
chapter: 3
pages: 25-26
---

# Ch 3 — A Dialogue on Virtualization

> [!abstract] One-sentence summary
> The **peach analogy**: one physical peach, many eaters, each feels like they have their own peach — because most of them are napping most of the time, and you can snatch the peach away during the nap. That's [[Virtualization]].

## The Analogy

- **Physical peach**: the real hardware (CPU, memory).
- **Eaters**: processes.
- **Virtual peaches**: the illusion given to each process that it owns the whole machine.
- **How**: since most eaters are napping (waiting on I/O, sleeping, idle), you can hand the peach to whoever is actually hungry *right now* — then take it back when they're done.

## Translated to the CPU

- One physical CPU, many processes.
- The OS rapidly switches among them.
- Each thinks it has its own CPU to use.
- The illusion is nearly perfect at human time scales.

This is [[Time Sharing]] — the fundamental trick behind CPU [[Virtualization]].

## Why It's Covered in a Dialogue

Dialogues in OSTEP are there to build *intuition* cheaply before the machinery comes. See [[OSTEP Book Conventions]]. The actual mechanics begin in [[Ch 4 — The Abstraction - The Process]].

## Related Notes

- [[Virtualization]]
- [[Ch 4 — The Abstraction - The Process]]
- [[MOC - CPU Virtualization]]
