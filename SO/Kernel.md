---
title: Kernel
tags:
  - concept
  - ostep
  - foundations
aliases:
  - OS Kernel
type: concept
introduced_in: Ch 6
---

# Kernel

> [!abstract] Definition
> The **kernel** is the privileged core of the operating system — the code that runs in [[User Mode vs Kernel Mode|kernel mode]] and has unrestricted access to the hardware. It handles system calls, interrupts, scheduling, memory management, file systems, and device drivers.

## What Qualifies as "Kernel"

Not every piece of the OS is in the kernel. A typical layout:

- **Kernel** — runs in kernel mode, always resident.
  - Scheduler, [[Process]] / [[Thread]] management.
  - [[Address Space]] / [[Virtual Memory]] machinery.
  - [[File System]] core.
  - [[Device Driver|Device drivers]] (in monolithic kernels; some drivers live in user space in microkernels).
  - Interrupt and [[Trap]] handlers.
- **Userspace OS code** — runs in user mode.
  - Shell, utilities, system services (init, logging, DNS resolver).
  - GUI, window manager, web browser, apps.

## Kernel Architectures

| Style | What's in the kernel | Examples |
|---|---|---|
| **Monolithic** | Everything: drivers, FS, networking | Linux, BSD, traditional UNIX |
| **Microkernel** | Just IPC, scheduling, primitive memory. Drivers/FS in user space. | MINIX, L4, QNX |
| **Hybrid** | Hard to classify; monolithic-ish with user-space services | macOS (XNU), Windows NT |

> [!tip] Trade-off
> Monolithic = fast (no IPC crossings). Microkernel = robust (driver crash doesn't kill the system). Most commercial OSes picked monolithic for speed, then added robustness by other means (driver signing, user-space drivers for slow devices, etc.).

## What Lives on the Kernel Stack

Each process has its own **kernel stack** used when the kernel runs on that process's behalf (during a syscall or interrupt). On a [[Trap]], user registers are pushed here.

## Not the Whole OS

People sometimes say "the kernel" when they mean "the OS". OSTEP is careful: the **OS** includes userspace services and tools; the **kernel** is just the privileged core.

## Related Notes

- [[Operating System]] — umbrella term.
- [[User Mode vs Kernel Mode]].
- [[System Call]], [[Trap]], [[Interrupt]].
- [[Limited Direct Execution]].
