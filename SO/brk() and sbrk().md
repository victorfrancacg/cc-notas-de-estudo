---
title: brk() and sbrk()
tags:
  - concept
  - ostep
  - memory
  - syscall
aliases:
  - brk
  - sbrk
  - Program Break
type: concept
introduced_in: Ch 14
---

# `brk()` and `sbrk()`

> [!abstract] Definition
> `brk` and `sbrk` are the classic UNIX system calls that **resize a process's [[Heap (Runtime)|heap]] segment** by moving the **program break** — a single pointer marking the heap's end. They are the low-level substrate underneath the C library's [[malloc()]] / [[free()]].

## The Program Break

A process's address space (in the simple model) has the heap immediately above the [[Code Segment|code]] / data segments. The end of the heap is the **break**.

```
   +---------------+  ← high addresses
   |  stack        |
   |  ...          |
   |  (free)       |
   +---------------+  ← break (heap end) ← brk/sbrk move this
   |  heap         |
   +---------------+
   |  code/data    |
   +---------------+  ← low addresses
```

## API

```c
#include <unistd.h>

int    brk(void *addr);      // set break to absolute address
void  *sbrk(intptr_t incr);  // move break by incr bytes; return previous break
```

- **`brk(addr)`** — set the break to absolute `addr`. Heap grows to that point.
- **`sbrk(incr)`** — increment the break by `incr` bytes; returns the **old** break (a pointer into the newly available region). `sbrk(0)` returns the current break without changing it.

## Don't Call These Directly

> [!warning] Use `malloc`/`free`, not `brk`/`sbrk`
> The C library's allocator owns the program break. If you call `brk` yourself, you'll fight the allocator's bookkeeping and corrupt its free lists. The OSTEP author is explicit: *stick to `malloc`/`free`*.

## Why This Mechanism Is Limited

- Single break ⇒ the heap is one contiguous region. You cannot free a hole in the middle and return that range to the OS via `brk` — the break only moves at the end.
- Releasing memory by lowering the break works only if the topmost chunk is free.

These limitations are why modern allocators rely heavily on **[[mmap()|anonymous `mmap()`]]** for large allocations — `mmap`'d regions can be released independently with `munmap`, regardless of position.

## Historical Note

`sbrk` predates `mmap`. On early UNIX, it was the only way to grow the heap. Today most allocators use it for the small-allocation arena and `mmap` for large allocations and per-thread arenas.

## Related Notes

- [[Heap (Runtime)]] — the region these calls grow.
- [[malloc()]], [[free()]] — the user-facing API on top.
- [[mmap()]] — the modern alternative for large/anonymous regions.
- [[System Call]] — `brk` and `sbrk` are syscalls (or thin wrappers around one).
- [[Address Space]] — the layout these calls modify.
- [[Ch 14 — Interlude - Memory API]].
