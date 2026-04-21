---
title: Cooperative vs Preemptive Scheduling
tags:
  - concept
  - ostep
  - scheduling
  - mechanism
aliases:
  - Cooperative Scheduling
  - Preemptive Scheduling
  - Non-preemptive Scheduling
type: concept
introduced_in: Ch 6
---

# Cooperative vs Preemptive Scheduling

> [!abstract] Definition
> Two fundamentally different ways the OS regains the CPU from a running process:
> - **Cooperative** — the OS **trusts** the process to give up the CPU voluntarily (via syscall, yield, or illegal operation).
> - **Preemptive** — the OS **takes** the CPU back at any time using a [[Timer Interrupt]].

## Cooperative Scheduling

### How It Works

The OS runs only when a process:

- Makes a [[System Call]] (e.g., `read()`, `write()`, or an explicit `yield()`).
- Triggers an exception (divide by zero, page fault, illegal instruction).

Otherwise, the process has the CPU forever.

### Where It Was Used

- Classic Mac OS (System 9 and earlier).
- Xerox Alto.
- Early cooperative multitasking Windows (3.x).

### Fatal Weakness

> [!warning] Infinite loop ⇒ dead machine
> A single buggy or malicious process that never makes a syscall hangs the entire system. The **only** recovery is to reboot.

### Strengths

- Simpler: no timer hardware required.
- Lower overhead: no periodic interrupt.
- Fewer concurrency bugs inside the kernel (since switches happen at known points).

## Preemptive Scheduling

### How It Works

The OS sets up a [[Timer Interrupt]] at boot. On every tick:

1. Hardware traps into the kernel regardless of what user code is doing.
2. The scheduler decides: keep running, or [[Context Switch]] to another process.

### Where It's Used

- Every modern mainstream OS: Linux, macOS, Windows NT+, BSD, Android, iOS.

### Strengths

- Robust against runaway processes — rogue code is stopped within one tick.
- Enables fine-grained [[Time Sharing]] and responsive interactive systems.
- Works naturally with [[Round Robin]] and most fairness-oriented schedulers.

### Costs

- Requires a programmable timer and interrupt handling.
- Concurrency inside the kernel is harder — preemption can happen in the middle of sensitive sequences. Protection via disabling interrupts briefly or using locks.

## Side-by-Side

| Aspect | Cooperative | Preemptive |
|---|---|---|
| Control mechanism | Voluntary syscalls | Hardware [[Timer Interrupt]] |
| Robustness to runaway | Poor (reboot) | Good |
| Kernel concurrency | Simpler | Harder |
| Latency for interactive tasks | Unbounded | Bounded by tick rate |
| Typical era | Pre-1990s desktops | Modern systems |

## Historical Footnote

Windows finally got preemptive multitasking for applications in Windows 95 (partially) and NT (fully). The leap in responsiveness was night-and-day.

## Related Notes

- [[Timer Interrupt]] — what enables preemption.
- [[Limited Direct Execution]] — the broader framework.
- [[Context Switch]] — the act the scheduler triggers.
- [[Scheduler]] — the policy layer.
