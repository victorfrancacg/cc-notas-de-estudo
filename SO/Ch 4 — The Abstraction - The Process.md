---
title: Ch 4 — The Abstraction - The Process
tags:
  - ostep
  - chapter
  - part/virtualization
  - process
type: chapter
book: OSTEP
chapter: 4
pages: 27-38
---

# Ch 4 — The Abstraction: The Process

> [!abstract] One-sentence summary
> A **[[Process]]** is a running program, described by its [[Machine State]] — memory ([[Address Space]]), registers (including [[Program Counter|PC]] and [[Stack Pointer|SP]]), and I/O info (open files). The OS creates the illusion of many CPUs by time-sharing processes.

## Crux of the Problem

> [!example] How to Provide the Illusion of Many CPUs?
> Although there are only a few physical CPUs available, how can the OS provide the illusion of a nearly-endless supply?
>
> **Answer:** [[Time Sharing]] + [[Context Switch|context switches]], coordinated by [[Mechanism vs Policy|mechanisms and policies]].

## What Is a Process?

The **process** is the OS's abstraction of a running program. A program on disk is *lifeless* — just bytes. The OS breathes life into it by loading it into memory and starting its execution. That live, executing instance is a process.

- [[Process]] — the core note.

## Machine State — What Defines a Process?

At any instant, a process is described by the [[Machine State]] it can read or update:

| Component | What | Note |
|---|---|---|
| **Memory** | [[Address Space]] — code, static data, [[Heap (Runtime)\|heap]], [[Stack (Runtime)\|stack]] | The process's "world". |
| **Registers** | General-purpose + special | Including [[Program Counter\|PC]] (next instruction), [[Stack Pointer\|SP]], frame pointer. |
| **I/O information** | Open files, etc. | On UNIX: file descriptor table. |

## Process API (Preview — Full in [[Ch 5 — Interlude - Process API|Ch 5]])

Any modern OS offers roughly:

- **Create** — `fork()`, `exec()`, or double-click.
- **Destroy** — `kill()` / `exit()`.
- **Wait** — pause until another process finishes.
- **Miscellaneous control** — suspend, resume.
- **Status** — inspect state, runtime, owner.

## Process Creation (4.3)

How does a program on disk become a live process?

1. **Load** code + static data into the process's [[Address Space]].
   - **Eagerly** in simple OSes; **lazily** (on demand) in modern ones — enabled by [[Paging]] and [[Swap Space]].
2. Allocate a **run-time [[Stack (Runtime)|stack]]** and initialize it with `argc`/`argv`.
3. Allocate a **[[Heap (Runtime)|heap]]** (for `malloc`/`free`).
4. Set up **I/O** (UNIX: open stdin, stdout, stderr).
5. Jump to the **`main()`** entry point. Off it goes.

## Process States (4.4)

Three essential states:

```
      Scheduled
   +------------->+
[Ready]           [Running]
   +<-------------+
      Descheduled

[Running] --I/O: initiate--> [Blocked]
[Blocked] --I/O: done-------> [Ready]
```

See [[Process States]] for full semantics.

## Data Structures (4.5)

- **[[Process List]]** (or **task list**) — tracks all processes.
- **[[Process Control Block]]** (PCB) — per-process struct. In xv6, it's `struct proc`, holding:
  - register context (`struct context`: eip, esp, ebx, ...)
  - state (`UNUSED`, `EMBRYO`, `SLEEPING`, `RUNNABLE`, `RUNNING`, `ZOMBIE`)
  - PID, parent pointer
  - memory / kernel-stack pointers
  - open files (`struct file *ofile[NOFILE]`)
  - trap frame

> [!info] Extra states you'll meet
> **Initial** — being created. **Zombie** (a.k.a. final) — exited but parent hasn't `wait()`ed yet. See [[Zombie Process]].

## Summary

By the end of this chapter:

- Know that a process is the OS abstraction of a running program.
- Know its components (memory, registers, I/O).
- Know the three essential states and the transitions.
- Know the OS tracks processes via a [[Process List]] of [[Process Control Block|PCBs]].

The next chapters show how the OS actually does this — starting with [[Ch 5 — Interlude - Process API|the API]], then [[Ch 6 — Mechanism - Limited Direct Execution|the mechanism]], then [[Ch 7 — Scheduling - Introduction|the policy]].

## Related Notes

- [[Process]], [[Process States]], [[Machine State]], [[Process Control Block]], [[Process List]].
- [[Time Sharing]], [[Mechanism vs Policy]].
- [[MOC - CPU Virtualization]].
