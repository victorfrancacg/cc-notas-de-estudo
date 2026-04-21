---
title: Process List
tags:
  - concept
  - ostep
  - process
  - data-structure
aliases:
  - Task List
type: concept
introduced_in: Ch 4
---

# Process List

> [!abstract] Definition
> The **process list** (a.k.a. **task list**) is the kernel-maintained collection of every [[Process]] currently in the system. Each entry points to a [[Process Control Block]].

## What It Enables

- **Enumerate** all processes (`ps` reads from here).
- **Schedule** — the [[Scheduler]] walks the list to pick a runnable process.
- **Cleanup** — on process exit, the OS removes its PCB from the list.
- **Signaling / `wait()`** — find a PID and deliver signals or collect exit status.

## Typical Implementation Choices

| Shape | When |
|---|---|
| **Linked list** | Simple OSes (xv6). O(n) scan; fine at small scale. |
| **Array** | Fixed-max-process systems. Fast index by PID slot. |
| **Hash table** | Large systems; O(1) lookup by PID. |
| **Red-black tree** | Real-time or sorted-by-deadline schedulers. |
| **Per-CPU runqueues** | Linux CFS — partitioned for scalability. |

## Sub-structures

Often the kernel doesn't keep a single monolithic list — it keeps multiple *views* over the same processes:

- **Runnable queue** — ready-to-run processes only. Scheduler scans this.
- **Sleep queue** — blocked processes, keyed by wait channel.
- **Zombie queue** — exited processes awaiting their parent's [[wait()]] call.

These are different structures pointing to the same PCBs.

## Related Notes

- [[Process Control Block]] — what each entry holds.
- [[Scheduler]] — primary consumer.
- [[Process States]] — determines which sub-list a process is on.
