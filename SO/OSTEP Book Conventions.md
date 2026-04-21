---
title: OSTEP Book Conventions
tags:
  - ostep
  - meta
type: reference
---

# OSTEP Book Conventions

> [!abstract] What this note is
> A reference for the didactic devices OSTEP uses throughout — so you can recognize them while reading.

## The Four Boxes

| Box | Purpose | How it looks |
|---|---|---|
| **Crux of the Problem** | States the central question the chapter exists to answer. Appears near the start. | Shaded, labeled "THE CRUX OF THE PROBLEM". |
| **Aside** | Tangentially-relevant background — interesting, non-essential. | Shaded, labeled "ASIDE: ...". |
| **Tip** | A general lesson that applies broadly (not chapter-specific). | Shaded, labeled "TIP: ...". |
| *(Figures / code)* | Real C code, not pseudocode. Runnable. | Numbered figures. |

> [!tip] Use the cruces as an exam-prep index
> At the end of the book, there's an index of every **crux**. Skim it — if you can state the crux *and* its answer for every chapter, you've internalized the book.

## The Dialogues

Mini-chapters framed as a conversation between **Professor** and **Student**. Used for:

- Introducing a major section (e.g., [[Ch 3 — A Dialogue on Virtualization]]).
- Reviewing a major section (e.g., Ch 11, Ch 24, Ch 34, Ch 46, Ch 47, Ch 51).
- A lighter tone that reinforces intuition without new mechanics.

## Real Code, Not Pseudocode

Examples (e.g., `cpu.c`, `mem.c`, `threads.c`, `io.c`) are real programs you can compile and run. The authors host them at:

- https://github.com/remzi-arpacidusseau/ostep-code

## Homeworks: Two Flavors

- **Simulation** — small Python simulators that model a subsystem (schedulers, page replacement, …). You can give them a seed to generate infinite practice problems, and ask them to solve the problems for self-check.
- **Real code** — write actual C against a UNIX-ish system.

Simulator repository:

- https://github.com/remzi-arpacidusseau/ostep-homework

## Projects

Systems programming projects (some targeting `xv6`, MIT's teaching OS):

- https://github.com/remzi-arpacidusseau/ostep-projects

## Related Notes

- [[MOC - OSTEP]] — the index.
- [[Ch 1 — A Dialogue on the Book]] — where the conventions are introduced.
