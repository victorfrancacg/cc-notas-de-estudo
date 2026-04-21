---
title: Context Switch
tags:
  - concept
  - ostep
  - mechanism
  - scheduling
type: concept
introduced_in: Ch 6
---

# Context Switch

> [!abstract] Definition
> A **context switch** is the kernel routine that saves the state of the currently running process and restores the state of another so that the new process can resume execution. It is the mechanism that makes [[Time Sharing]] of the CPU possible.

## The Minimum Job

Save enough of "process A's world" so that, if we restore it later, A resumes *as if nothing happened*:

- Kernel registers (callee-saved regs, kernel SP, kernel PC).
- User registers (already saved to A's kernel stack by the hardware on entry).
- Pointer to A's [[Address Space]] (page table base register — so the MMU sees the right mappings).

Then load B's equivalent state.

## Two Layers of Saving

| Layer | Who saves | Where |
|---|---|---|
| **User registers** | Hardware, on trap | A's kernel stack |
| **Kernel registers** | Software — the `switch` routine | [[Process Control Block]] |

Both must be restored for B to resume properly.

## The xv6 switch() in Essence

```asm
; void swtch(struct context *old, struct context *new);
swtch:
    ; save old context (callee-saved registers + sp + pc)
    mov  [old], esp
    mov  [old+4], ebx
    ; ...etc...

    ; load new context
    mov  esp, [new]
    mov  ebx, [new+4]
    ; ...etc...
    ret     ; returns to new's saved PC
```

Subtle: you **enter** this routine in A's context and **return** in B's context — because the stack was switched mid-function.

## When Context Switches Happen

- On a [[Timer Interrupt]], when the scheduler decides to preempt.
- On a blocking syscall (process goes to [[Process States|Blocked]] state).
- On an explicit `yield()` (cooperative).
- When a process exits.

## Cost

| Era | Cost |
|---|---|
| Mid-1990s | ~6 µs |
| 2020s | sub-microsecond on fast CPUs |

But the *real* cost of a context switch includes hidden things: **cache pollution** — the new process's working set isn't in the cache, so the next few hundred instructions are slower. This hidden cost can dwarf the direct register-saving cost.

## What's *Not* Switched

- **Physical memory contents** — belong to whichever process's pages are in RAM; paging is orthogonal.
- **Hardware caches** — they cool down naturally; switching them is not possible (and mostly not desirable — see [[Cache Affinity]] on why you *want* cache contents preserved across switches to the same process).

## Related Notes

- [[Process]], [[Process Control Block]].
- [[Timer Interrupt]], [[Trap]].
- [[Limited Direct Execution]].
- [[Scheduler]] — decides *whether* to switch.
- [[Cache Affinity]] — why repeated switches to a specific CPU help.
- [[Ch 6 — Mechanism - Limited Direct Execution]].
