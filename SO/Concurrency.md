---
title: Concurrency
tags:
  - concept
  - ostep
  - foundations
  - concurrency
aliases:
  - Concurrent
type: concept
introduced_in: Ch 2
---

# Concurrency

> [!abstract] Definition
> **Concurrency** refers to the set of problems that arise when many things happen *at the same time* within a program — whether because the OS is juggling processes, or because a program itself uses multiple threads sharing memory.

It's the second of [[Three Easy Pieces|the three pieces]] and the focus of Part II of [[MOC - OSTEP|OSTEP]].

## Where It Comes From

Concurrency first arose **inside the OS** — the OS is itself a massive concurrent program, juggling many processes, interrupts, and I/O devices. But now, thanks to **[[Multi-threaded Program|multi-threaded]]** user programs, every developer faces the same problems.

## The Canonical Broken Example

```c
volatile int counter = 0;

void *worker(void *arg) {
    for (int i = 0; i < loops; i++) {
        counter++;          // <-- looks atomic, isn't
    }
    return NULL;
}
```

Launch two threads. Expected final counter with `loops = 100000`: **200000**. Actual: **143012**, **137298**, …, different each run.

### Why?

`counter++` compiles to roughly **three instructions**:

1. `load` counter from memory into a register.
2. `increment` the register.
3. `store` the register back to memory.

The scheduler can preempt between any of these. If Thread A loads, Thread B loads (same old value), both increment, both store — one increment is lost. This is a **[[Race Condition]]**.

> [!warning] The root cause is non-atomicity
> High-level operations that look indivisible often aren't. See [[Atomicity]].

## Crux of the Problem

> [!example] How to Build Correct Concurrent Programs
> When multiple threads share memory and the scheduler can interrupt anywhere, what primitives from hardware and OS do we need to write code that is **correct** and **performant**?

Part II of the book builds the primitives for this: [[Lock]]s, [[Condition Variable]]s, [[Semaphore]]s, and discusses classic problems ([[Producer-Consumer Problem]], [[Dining Philosophers]]) and common bugs ([[Deadlock]]).

## Concurrency vs Parallelism

> [!info] A useful distinction (not always made in OSTEP)
> - **Concurrency** — *dealing with* many things at once (a property of the program's structure).
> - **Parallelism** — *doing* many things at once (a property of the execution — requires multiple cores).
>
> A single-core machine can be concurrent (interleaved) but not parallel. The concurrency problems are the same either way.

## Related Notes

- [[Process]], [[Thread]] — the agents that concurrency coordinates.
- [[Race Condition]], [[Atomicity]] — why concurrency is hard.
- [[Multiprogramming]] — historical origin.
- [[Virtualization]] — concurrency arises *inside* the virtualization machinery.
- [[Operating System]] — the biggest concurrent program you'll see.
