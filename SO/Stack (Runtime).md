---
title: Stack (Runtime)
tags:
  - concept
  - ostep
  - memory
  - address-space
aliases:
  - Stack
  - Call Stack
  - Runtime Stack
  - Process Stack
type: concept
introduced_in: Ch 4
detailed_in: Ch 13
---

# Stack (Runtime)

> [!abstract] Definition
> The **stack** is the region of a [[Process|process's]] [[Address Space]] that holds **call frames** — locals, function arguments, return addresses, saved registers — for the active function-call chain. Compiler-managed: pushed on call, popped on return. Conventionally placed at the top of the address space and **grows downward** toward the [[Heap (Runtime)|heap]].

## What Lives on the Stack

Each function call pushes a **stack frame** containing:

- **Local variables** of the function.
- **Function arguments** (some passed in registers, some on the stack — calling-convention specific).
- **Return address** — where to resume in the caller after this function returns.
- **Saved registers** — caller- or callee-saved per the ABI.
- **Frame pointer** (sometimes) — pointer to the caller's frame, easing debugger walks.

When the function returns, its frame is popped; the memory becomes available for the next call.

## Direction of Growth

```
   0KB  ┌──────────────────┐
        │ Code             │
   1KB  ├──────────────────┤
        │ Heap             │  grows ↓
        │     ↓            │
        │  (free)          │
        │     ↑            │
        │ Stack            │  grows ↑   (toward smaller addresses)
  16KB  └──────────────────┘
```

> [!info] "Grows down" = lower addresses
> On most modern CPUs (x86, ARM) the stack pointer **decreases** on push. The stack starts at a high virtual address and creeps toward the heap as call depth grows.

## Stack vs Heap

| Stack | Heap |
|---|---|
| Allocated implicitly (compiler emits push/pop) | Allocated explicitly (`malloc`/`new`) |
| Freed automatically on return | Freed manually (`free`/`delete`) or by GC |
| LIFO order, fixed-size frames | Arbitrary order, variable-size blocks |
| Cheap (just SP arithmetic) | Expensive (allocator bookkeeping) |
| Limited size (often ~8MB) | Limited only by available virtual memory |
| One per [[Thread]] | Shared by all threads of a process |

## Common Stack Bugs

> [!warning] Stack overflow
> Infinite recursion or huge stack allocations push past the stack limit. Modern kernels place a **guard page** below the stack so that the next access traps and the process dies cleanly (segfault).

> [!warning] Returning a pointer to a local
> ```c
> int *bad() {
>     int x = 7;
>     return &x;   // dangling — x lives on the stack frame
> }                // frame popped here; pointer now points at garbage
> ```
> Returning a pointer to a stack local is undefined behaviour and a classic source of bugs.

> [!warning] Buffer overflow → control-flow hijack
> Writing past a stack buffer can overwrite the saved return address. This is the canonical stack-smashing attack; modern systems mitigate via stack canaries, ASLR, and non-executable stacks.

## Threads and Multiple Stacks

A multi-threaded program has **one stack per thread** (allocated via `pthread_create` or equivalent). The address-space layout becomes more complex — there is no longer a single "stack at the top of memory" but several stacks, often allocated as separate regions via `mmap`. See [[Thread]].

## Stack and Memory Virtualization

Stack memory is **virtual** like everything else in the address space. When a thread is created, its stack region is mapped lazily — physical pages are allocated only as the stack pointer grows into new pages. The illusion of "8MB of stack" doesn't cost 8MB of physical RAM up front.

## Related Notes

- [[Address Space]] — the stack is one of its three canonical regions.
- [[Heap (Runtime)]] — the stack's neighbor and opposite.
- [[Code Segment]] — static instructions; doesn't grow.
- [[Process]] — every process has at least one stack.
- [[Thread]] — each thread has its own stack.
- [[Context Switch]] — saves/restores the stack pointer along with other registers.
- [[Machine State]] — the stack pointer is part of it.
