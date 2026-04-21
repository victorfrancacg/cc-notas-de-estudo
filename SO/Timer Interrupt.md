---
title: Timer Interrupt
tags:
  - concept
  - ostep
  - mechanism
  - hardware
aliases:
  - Preemption Timer
type: concept
introduced_in: Ch 6
---

# Timer Interrupt

> [!abstract] Definition
> A **timer interrupt** is a hardware [[Interrupt]] fired periodically by a programmable timer chip. Every tick, the currently running process is paused and the OS's timer interrupt handler runs — giving the kernel a chance to re-evaluate which process should be on the CPU.

## Why It's Essential

Without a timer:

- A rogue or buggy process can infinite-loop forever, and the OS can't take the CPU back.
- Only way to recover: **reboot**.

With a timer:

- The OS regains control every ~1–10 ms, *no matter what* user code is doing.
- [[Cooperative vs Preemptive Scheduling|Preemptive]] scheduling becomes possible.
- The illusion of [[Time Sharing]] is realizable.

## How It Works

At boot:

1. Kernel registers its **timer interrupt handler** in the [[Trap Table]].
2. Kernel programs the timer chip to fire every X ms.
3. Kernel starts the timer (privileged instruction).

At runtime, on each tick:

1. Hardware fires the interrupt — saves user registers, switches to kernel mode.
2. Handler runs. It usually:
   - Updates kernel time-keeping.
   - Decrements the running process's time slice.
   - If the slice is up, invokes the [[Scheduler]] to pick another process.
   - Performs a [[Context Switch]] if a switch is warranted.
3. Return-from-trap to user mode — same process if no switch, different one if switched.

## Tick Rate

Historically: 100 Hz (10 ms). Modern Linux: **tickless** kernels — the timer is reprogrammed dynamically to fire only when needed, saving power on idle machines.

Smaller ticks = finer scheduling granularity but more overhead.

## Privileged Operation

Starting, stopping, or reprogramming the timer is a **privileged** operation. User code cannot disable the timer — this is crucial, because a user-disabled timer would defeat preemption.

## Related Notes

- [[Interrupt]] — parent concept.
- [[Trap]], [[Trap Table]] — machinery shared.
- [[Context Switch]] — what the handler often triggers.
- [[Scheduler]] — decides after the tick.
- [[Limited Direct Execution]] — timer is a key ingredient.
- [[Cooperative vs Preemptive Scheduling]] — timer is what enables preemption.
