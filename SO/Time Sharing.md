---
title: Time Sharing
tags:
  - concept
  - ostep
  - virtualization
  - scheduling
aliases:
  - Time-Sharing
  - Timesharing
type: concept
introduced_in: Ch 2
---

# Time Sharing

> [!abstract] Definition
> **Time sharing** is the strategy of letting many actors share one resource by **rapidly switching** the resource among them over time, so each actor *seems* to have continuous access.

## Applied to the CPU

This is the foundational technique for **virtualizing the CPU**. With one CPU and 100 processes, the OS:

1. Runs process A for a few milliseconds.
2. [[Context Switch|Switches]] to process B.
3. Runs B for a few milliseconds.
4. Switches to C, and so on.

Because switches happen on the microsecond scale while humans perceive change on the hundred-millisecond scale, everything *appears* to run at once.

## What Time Sharing Requires

| Ingredient | Why |
|---|---|
| [[Context Switch]] | Mechanism to save one process's state and restore another's. |
| [[Timer Interrupt]] | So the OS can forcibly take the CPU back (else a misbehaving process hogs forever). |
| [[Scheduler]] | Policy to decide *which* process runs next. |
| [[Limited Direct Execution]] | Ensures the switch is safe despite running user code directly. |

## Historical Root

The term dates to the 1960s CTSS / Multics era, when "time sharing" *was* the marquee OS feature: multiple users interacting with one machine, instead of submitting jobs in a [[Batch Processing|batch]]. Today, time sharing is so ubiquitous we don't even name it.

## Time Sharing vs Space Sharing

| | Time sharing | [[Space Sharing]] |
|---|---|---|
| Unit divided | Time | The resource itself |
| CPU example | Each process gets slices of time | (No — CPUs don't split this way) |
| Disk example | (Possible but rare for raw blocks) | Each file occupies distinct blocks |
| Memory example | (Too slow to swap whole RAM) | Each process gets distinct pages |

> [!tip] Which one for which resource?
> Use **time sharing** when the resource can't be meaningfully subdivided (e.g., a CPU core) or when sharing is infrequent. Use **space sharing** when partitioning is cheap (e.g., disk blocks, memory pages).

## Related Notes

- [[Virtualization]] — time sharing *is* how CPU virtualization works.
- [[Context Switch]], [[Scheduler]], [[Timer Interrupt]] — the machinery.
- [[Multiprogramming]] — the idea behind time-sharing CPU use.
- [[Space Sharing]] — the contrasting strategy.
