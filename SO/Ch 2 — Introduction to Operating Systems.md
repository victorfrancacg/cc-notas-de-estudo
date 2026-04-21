---
title: Ch 2 — Introduction to Operating Systems
tags:
  - ostep
  - chapter
  - part/virtualization
  - foundations
type: chapter
book: OSTEP
chapter: 2
pages: 3-21
---

# Ch 2 — Introduction to Operating Systems

> [!abstract] One-sentence summary
> An [[Operating System]] is software that virtualizes physical resources (CPU, memory, disk) into easier-to-use virtual forms, coordinates concurrent actors, and persists data — all while acting as a **resource manager**.

## The Von Neumann Model (baseline)

A running program does one simple thing: **fetch → decode → execute**, instruction after instruction. Everything the OS provides is built on top of — and tries to hide the complications of — this basic cycle.

See: [[Von Neumann Model]].

## What an OS Does — Three Big Jobs

> [!example] Crux #1: How to Virtualize Resources
> *How does the OS take physical hardware and turn it into a more general, powerful, easy-to-use virtual form?* The answer runs through the whole first part of the book.

### 2.1 Virtualizing the CPU

- Running `./cpu A` repeatedly prints `A` once per second — boring.
- Running `./cpu A & ./cpu B & ./cpu C & ./cpu D &` gets *four* letters interleaved, all "running at once" even on a single CPU.
- The OS creates the **illusion** of many virtual CPUs from one. This is [[Time Sharing]].
- Requires **APIs** (how users start/stop/control programs) and **policies** (which to run when). See [[Mechanism vs Policy]].

### 2.2 Virtualizing Memory

- Physical memory = an array of bytes, addressed by integers.
- Run two copies of a `./mem` program that `malloc`s a pointer, each reports address `0x200000` — *the same address!*
- Yet each increments *its own* counter independently. How?
- Each process sees its own [[Address Space]]; the OS maps these virtual addresses onto different physical locations.
- Intuition: every process *acts as if it owns all of memory*.

### 2.3 Concurrency

> [!example] Crux #2: How to Build Correct Concurrent Programs
> When threads share memory and the scheduler can interrupt anywhere, how do we write correct code?

- Example: two threads, each loop incrementing a shared `counter`.
- Expected with `loops = 100000`: final counter = 200000.
- Observed: 143012, 137298, … different each run, almost never 200000.
- Why? `counter++` is **not atomic**: load → increment → store. Interleaving breaks it.
- See [[Concurrency]], [[Atomicity]].

### 2.4 Persistence

> [!example] Crux #3: How to Store Data Persistently
> How to reliably store data on volatile-free media, with performance, in the face of crashes?

- DRAM is **volatile** — lose it on power loss or crash.
- Hardware: HDDs, SSDs. Software: the [[File System]].
- Unlike CPU/memory, files are **not virtualized per-process** — they're meant to be shared among processes (e.g., editor creates `main.c`, compiler reads it, shell runs the binary).
- System calls: `open()`, `write()`, `close()` enter the file system.
- Techniques for crash safety: **journaling**, **copy-on-write**.

## 2.5 Design Goals

> [!tip] The OS is full of tradeoffs
> Every OS design decision balances these goals against each other. Few can be fully maximized simultaneously.

See [[OS Design Goals]]. In brief:

- **Abstraction** — the foundational goal; everything else rests on it.
- **Performance** — minimize overheads (space and time).
- **Protection / Isolation** — processes can't harm each other or the OS.
- **Reliability** — the OS must not fail; if it does, everything above it falls too.
- **Energy efficiency**, **security**, **mobility** — modern concerns.

## 2.6 A Brief History

| Era | Key idea |
|---|---|
| Early (libraries) | OS = collection of shared routines. [[Batch Processing]] with human operator. |
| Protection | [[System Call]] invented (Atlas). [[User Mode vs Kernel Mode]]. [[Trap]]. |
| Minicomputers / [[Multiprogramming]] | Load many jobs, switch on I/O. Born: memory protection, concurrency awareness. UNIX (Thompson/Ritchie/Bell Labs). |
| Personal computers | Early PCs (DOS, classic Mac OS) *lost* prior lessons — no memory protection, cooperative scheduling. |
| Modern | macOS (UNIX core), Windows NT, Linux (Torvalds), Android. Good ideas re-emerge. |

> [!info] Asides in this chapter
> - **The Importance of UNIX** — the unifying influence; pipes, C language, open-source seeds.
> - **And Then Came Linux** — Torvalds built a UNIX-like kernel without touching original code, avoiding legal issues that slowed the other BSDs.

## 2.7 Summary — What the Book *Won't* Cover

- Networking (take a networking class).
- Graphics (take a graphics class).
- Deep security (take a security class).

## Related Notes

- [[Operating System]] — definition and roles, expanded.
- [[Virtualization]] — the core technique of Part I.
- [[Concurrency]] — Part II.
- [[Persistence]] — Part III.
- [[Mechanism vs Policy]] — this separation recurs everywhere.
- [[Multiprogramming]] — historical driver of most OS features.
