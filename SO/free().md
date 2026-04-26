---
title: free()
tags:
  - concept
  - ostep
  - memory
  - api
  - libc
aliases:
  - free
type: concept
introduced_in: Ch 14
---

# free()

> [!abstract] Definition
> `free(void *ptr)` is the C library call that **returns a [[malloc()]]'d block to the heap allocator** for reuse. The user only passes the pointer — the allocator tracked the size in a header word just before `ptr`.

## Signature

```c
#include <stdlib.h>

void free(void *ptr);
```

Calling `free(NULL)` is **safe** and a no-op (since C89). No need to guard with `if (p)`.

## Why You Don't Pass the Size

The allocator stores per-block metadata (size, free/used flag, free-list pointers) in a small header right before the user-visible memory. `free(p)` walks back one header from `p`, reads the size, and links the block onto the free list.

```
       ┌── header ──┐
       │ size │ ... │
       ├─────────────┤
p  ──→ │ user bytes  │
       │             │
       └─────────────┘
```

That's also why **invalid frees** (passing a pointer that isn't a `malloc`'d address) are dangerous: the allocator reads garbage as a header.

## What Happens After `free`

- The block joins the free list. Future `malloc` calls may reuse it.
- The block is *not* automatically returned to the OS. Modern allocators only release pages back via `munmap` for large blocks or after a heuristic.
- Adjacent free blocks may be **coalesced** to combat fragmentation — see [[Free-Space Management]].

> [!info] The pointer still points somewhere
> After `free(p)`, `p` itself is unchanged — it still holds the old address. The allocator is now free to hand that memory out elsewhere. Reading or writing through `p` becomes a **[[Dangling Pointer]]** bug.

## Common `free` Bugs

- **[[Double Free]]** — calling `free(p)` twice. Corrupts the allocator's internal state.
- **[[Memory Leak]]** — *not* calling `free`. Memory grows unbounded.
- **Invalid free** — passing a stack address, an interior pointer, or a never-allocated pointer.
- **Use-after-free** — see [[Dangling Pointer]].

> [!tip] Defensive idiom
> ```c
> free(p);
> p = NULL;   // makes future use-after-free explode immediately, not silently
> ```

## OS-Side Reclamation

On process exit, the kernel reclaims **all** of the process's pages — including leaked heap blocks. Short-lived programs that don't `free` won't leak permanently because of this. But long-running programs (servers, OS itself) must be disciplined.

## Related Notes

- [[malloc()]] — the matching allocator.
- [[Heap (Runtime)]] — what `free` returns memory to.
- [[Free-Space Management]] — how the allocator carves and merges blocks (Ch 17).
- [[Memory Leak]], [[Dangling Pointer]], [[Double Free]] — `free`-related bugs.
- [[mmap()]] — large allocations may go directly to `munmap`.
