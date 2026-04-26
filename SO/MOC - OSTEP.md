---
title: MOC - OSTEP (Operating Systems: Three Easy Pieces)
tags:
  - moc
  - ostep
  - operating-systems
aliases:
  - OSTEP
  - OSTEP Book MOC
  - Operating Systems Three Easy Pieces
type: moc
book: Operating Systems - Three Easy Pieces
authors: Remzi H. Arpaci-Dusseau, Andrea C. Arpaci-Dusseau
edition: Version 1.10 (October 2023)
---

# MOC — Operating Systems: Three Easy Pieces

> [!abstract] The Book's Big Idea
> An operating system is responsible for three main things: **[[Virtualization]]**, **[[Concurrency]]**, and **[[Persistence]]**. These are the "three easy pieces" that organize the entire book.

## The Three Pieces

- [[MOC - CPU Virtualization]] — Part I (CPU side): the process abstraction, process API, limited direct execution, scheduling. *(Ch 1–11, **done**)*
- [[MOC - Memory Virtualization]] — Part II: address spaces, paging, TLBs, swapping. *(Ch 12–24, **in progress** — Ch 12–20 done)*
- Concurrency — Part III: threads, locks, condition variables, semaphores. *(Ch 25–34, pending)*
- Persistence — Part IV: I/O devices, disks, RAID, file systems, journaling, distributed systems. *(Ch 35–51, pending)*

## Progress Tracker

| Part | Chapters | Status | Notes created |
|------|---------:|:------:|:---:|
| CPU Virtualization | 1–11 | **Done** | ~70 |
| Memory Virtualization | 12–20 | Done so far | ~40 (in Part II) |
| Memory Virtualization | 21–24 | Pending | — |
| Concurrency | 25–34 | Pending | — |
| Persistence | 35–51 | Pending | — |

> [!note] Session state (as of 2026-04-26)
> **Part I (CPU Virtualization) complete** through Ch 11. **Part II (Memory Virtualization)** progressing — Ch 12–20 done this session: dialogues, address spaces, memory API, address translation, segmentation, free-space management, paging, TLBs, smaller tables. Next session resumes at [[Ch 21 — Beyond Physical Memory - Mechanisms|Ch 21]] (swap space and page faults).

## Cross-Cutting Concepts

> [!tip] The Three Key Themes
> Wherever you go in the book, ask: is this a **virtualization** story, a **concurrency** story, or a **persistence** story? Every problem ultimately maps to one of these.

- [[Virtualization]] — making one physical resource look like many.
- [[Concurrency]] — coordinating multiple simultaneous actors sharing state.
- [[Persistence]] — storing data reliably across crashes and time.

## How to Navigate This Vault

- **MOC notes** (`MOC - *`) are indexes. Start from here, follow wikilinks.
- **Concept notes** are atomic (one idea per note). They link to each other densely — the graph view is the real map.
- **Problem notes** walk through classic problems (e.g., [[Producer-Consumer Problem]], [[Dining Philosophers]]).
- **Algorithm notes** cover specific algorithms (e.g., [[FIFO Scheduling]], [[Round Robin]], [[MLFQ]]).

## Study Technique

> [!tip] "Crux of the Problem"
> OSTEP organizes every chapter around a **crux**: the central question the chapter exists to answer. When reviewing, always ask: *what was the crux? what was the answer?* If you can restate both in one sentence, you own the chapter.

See also: [[OSTEP Book Conventions]] for the didactic devices used throughout (asides, tips, cruces, dialogues).
