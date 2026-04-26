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

- **Part I — Virtualization** *(Ch 1–24, **complete**)*. Two halves indexed by sub-MOCs:
  - [[MOC - CPU Virtualization]] — CPU half: process, process API, LDE, scheduling. *(Ch 1–11)*
  - [[MOC - Memory Virtualization]] — Memory half: address spaces, paging, TLBs, swapping, replacement, real-system case studies. *(Ch 12–24)*
- **Part II — Concurrency** *(Ch 25–34)*: threads, locks, condition variables, semaphores. *(pending)*
- **Part III — Persistence** *(Ch 35–51)*: I/O devices, disks, RAID, file systems, journaling, distributed systems. *(pending)*

## Progress Tracker

| Section | Chapters | Status | Notes created |
|------|---------:|:------:|:---:|
| Part I — CPU Virtualization | 1–11 | **Done** | ~70 |
| Part I — Memory Virtualization | 12–24 | **Done** | ~70 |
| Part II — Concurrency | 25–34 | Pending | — |
| Part III — Persistence | 35–51 | Pending | — |

> [!note] Session state (as of 2026-04-26)
> **Part I — Virtualization — complete.** Ch 21–24 wrap memory virtualization: swap mechanisms, replacement policies (FIFO/LRU/Clock/Optimal), VAX/VMS + Linux case studies, closing dialogue. Next session: Part II (Concurrency) — opens with [[Ch 25 — A Dialogue on Concurrency|Ch 25]], threads and the basics of shared-state coordination.

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
