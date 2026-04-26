---
title: Segmentation Fault
tags:
  - concept
  - ostep
  - memory
  - bug
aliases:
  - Segfault
  - SIGSEGV
type: concept
introduced_in: Ch 14
---

# Segmentation Fault

> [!abstract] Definition
> A **segmentation fault** (segfault, SIGSEGV) is the OS terminating a process for accessing memory it isn't allowed to touch — an unmapped page, a read-only page being written, or kernel memory from user mode. The hardware traps; the OS delivers `SIGSEGV`; the process usually dies with a core dump.

## How It Happens

The hardware ([[MMU]]) checks every memory access against the process's [[Page Table|page-table]] permissions. If the access fails:

1. The MMU raises a fault.
2. The CPU traps into the kernel ([[Trap]], [[Trap Table]]).
3. The kernel inspects the faulting address and the operation (read/write/execute).
4. If it's a legitimate-but-unresolved access (page swapped to disk, copy-on-write, demand paging), the kernel handles it transparently.
5. Otherwise the kernel sends `SIGSEGV` to the process. Default handler: dump core and terminate.

## Classic Causes

- **NULL dereference** — `*p` where `p == NULL`. Page 0 is intentionally unmapped so this trap is **clean** rather than silent corruption.
- **[[Dangling Pointer|Use-after-free]] on an unmapped page** — sometimes; if the page got returned to the OS via `munmap`.
- **Stack overflow** — recursion or huge locals push past the guard page placed below the stack.
- **Buffer overflow into a read-only or unmapped region** — see [[Buffer Overflow]].
- **Writing to a read-only mapping** — e.g., trying to modify a string literal:
  ```c
  char *s = "hello";   // points into .rodata
  s[0] = 'H';          // segfault: page is read-only
  ```
- **Executing data on a non-executable page** — typical W^X enforcement.

## Why Segfaults Are *Good*

> [!tip] Better to crash than to corrupt
> Without [[Memory Virtualization|virtual memory]] and [[Isolation (OS)|isolation]], a wild pointer would silently scribble over another process's data — or worse, the kernel's. The MMU turns those bugs into immediate, debuggable crashes. The OS is doing its job.

## Debugging

- **gdb**: run inside it; the segfault stops execution at the faulting instruction.
- **Core dumps**: enable with `ulimit -c unlimited`; analyze with `gdb prog core`.
- **AddressSanitizer**: catches many would-be segfaults earlier with richer diagnostics.

## "Segmentation" — Why That Word?

The name is historical, dating to systems that organized memory in segments (see [[Segmentation|Ch 16]]). Modern paged systems don't actually use segments, but the trap signal kept the name.

## Related Notes

- [[Memory Virtualization]] — what makes segfaults possible (and protective).
- [[MMU]], [[Page Table]] — the enforcers.
- [[Dangling Pointer]], [[Buffer Overflow]] — common segfault sources.
- [[Isolation (OS)]] — the design principle behind it.
- [[Trap]] — how the fault enters the kernel.
- [[Ch 14 — Interlude - Memory API]].
