---
title: malloc()
tags:
  - concept
  - ostep
  - memory
  - api
  - libc
aliases:
  - malloc
type: concept
introduced_in: Ch 14
---

# malloc()

> [!abstract] Definition
> `malloc(size_t size)` is the C standard-library call that **allocates `size` bytes** on the [[Heap (Runtime)|heap]] and returns a pointer to them, or `NULL` on failure. It is **library code**, not a system call — it manages a region the OS already gave the process via [[brk() and sbrk()|brk]] or [[mmap()]].

## Signature & Usage

```c
#include <stdlib.h>

void *malloc(size_t size);
```

```c
int    *p1 = malloc(sizeof(int));            // one int
int    *p2 = malloc(10 * sizeof(int));       // array of 10
double *p3 = malloc(sizeof(double));         // one double
char   *s  = malloc(strlen(src) + 1);        // +1 for '\0'
```

> [!tip] Use `sizeof()` for portability
> Hard-coding `8` for a `double` works on 64-bit but not always elsewhere. `sizeof(double)` is computed at compile time and gives the right number on every platform.

## What `malloc` Does Not Do

- **Doesn't zero the memory.** The bytes are whatever was there before. Use [[calloc()]] (or `memset`) if you need zeros — see [[Uninitialized Memory Read]].
- **Doesn't track size for the user.** C has no `sizeof_malloc(p)`. The size lives in allocator metadata, not visible.
- **Doesn't grow allocations.** Use [[realloc()]] for that.
- **Doesn't fail loudly.** On out-of-memory, returns `NULL`. **Always check.**

## How `malloc` Actually Works

`malloc` sits on a **per-process heap arena** that is grown via:

- **[[brk() and sbrk()]]** for small/incremental needs (older convention).
- **[[mmap()]] (anonymous)** for large allocations and fresh arenas.

Within an arena, [[Free-Space Management|free-space management]] (Ch 17) carves the available bytes into blocks — using a **header word** just before the returned pointer to record the block's size and (often) free/used flag.

```
        ┌──── block header ────┐
        │ size │ flags │ ...   │
        ├───────────────────────┤
ptr ──→ │ user-visible bytes   │   ← what malloc returns
        │                      │
        └───────────────────────┘
```

This is why [[free()]] only needs the pointer: it walks back one header to find the size.

## Performance Notes

- Small allocations are usually fast (free-list lookup).
- Large allocations (`mmap` path) are slower but get returned to the OS on `free` — small allocations stay in the process's pool for reuse.
- Modern allocators (jemalloc, tcmalloc) are **thread-aware** — per-thread arenas reduce contention.

## Common Mistakes

- **Forgetting `+1` for null terminator** on string buffers ([[Buffer Overflow]]).
- **Using `sizeof(p)` instead of `sizeof(*p)`** when `p` is a pointer.
- **Not checking the return value.** `NULL` deref is a clean [[Segmentation Fault|segfault]] — better than silent corruption, but still a bug.
- **Casting unnecessarily in C.** `(int *) malloc(...)` is legal but redundant; the cast is required only in C++.

## Related Notes

- [[free()]] — the matching deallocator.
- [[calloc()]], [[realloc()]] — sibling allocators.
- [[Heap (Runtime)]] — where `malloc` allocates from.
- [[brk() and sbrk()]], [[mmap()]] — the OS-side mechanisms.
- [[Free-Space Management]] — how the allocator picks blocks (Ch 17).
- [[Memory Leak]], [[Dangling Pointer]], [[Double Free]] — bugs the API enables.
