---
title: Persistence
tags:
  - concept
  - ostep
  - foundations
  - persistence
aliases:
  - Persistent
  - Persistently
type: concept
introduced_in: Ch 2
---

# Persistence

> [!abstract] Definition
> **Persistence** is the problem of storing data so it survives **power loss**, **crashes**, and the passage of time. In the OS world, it's the job of the [[File System]] running on top of non-volatile hardware.

It's the third of [[Three Easy Pieces|the three pieces]] and the focus of Part III of [[MOC - OSTEP|OSTEP]].

## Why It's a Distinct Problem

- **DRAM is volatile.** When the power goes, everything is gone.
- Users care about *their data* — photos, code, essays. The machine can reboot; the data cannot be lost.
- Thus the OS must cooperate with **persistent hardware** (HDDs, SSDs) to store data reliably.

## Hardware + Software

| Layer | Examples |
|---|---|
| **Persistent hardware** | [[Hard Disk Drive\|HDD]], [[SSD]], flash |
| **Device driver** | Code that knows how to talk to a specific piece of storage hardware. |
| **File system** | OS component that organizes raw storage into **files** and **directories**. |

## A Minimal Example

```c
int fd = open("/tmp/file", O_WRONLY | O_CREAT | O_TRUNC, S_IRWXU);
write(fd, "hello world\n", 13);
close(fd);
```

Three system calls, three OS-level steps:

1. `open()` — ask the file system to create / open the file.
2. `write()` — buffer the bytes, arrange for them to reach disk.
3. `close()` — signal that no more writes are coming.

Underneath, the file system:

- Figures out *where* on disk the bytes go.
- Updates its own metadata (inodes, free-space maps).
- Issues I/O to the device driver.

## Persistence ≠ Virtualization

> [!warning] Files are shared, not virtualized per-process
> Unlike CPU or memory, the OS *intentionally* exposes files as **shared** so processes can cooperate: an editor writes `main.c`, a compiler reads it, the shell runs the output. Private per-process files would defeat the point.

## The Hard Parts

Writes can fail mid-flight (power cut, crash). To stay consistent, file systems use intricate techniques:

- **[[Journaling]]** — write intent to a log before the actual update; on crash, replay.
- **Copy-on-write** — never overwrite in place; write new blocks, then swap pointers atomically.
- **[[Checksums]]** — detect corruption from misdirected or lost writes.

## Related Notes

- [[Operating System]] — where the file system lives.
- [[File System]] — the deeper dive (covered in Part III).
- [[System Call]] — the entry point (`open`, `read`, `write`, `close`).
- [[Crash Consistency]] — the sub-problem most persistence techniques target.
- [[Virtualization]], [[Concurrency]] — the other two pieces.
