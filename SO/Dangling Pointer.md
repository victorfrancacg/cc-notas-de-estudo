---
title: Dangling Pointer
tags:
  - concept
  - ostep
  - memory
  - bug
aliases:
  - Use-After-Free
  - UAF
type: concept
introduced_in: Ch 14
---

# Dangling Pointer

> [!abstract] Definition
> A **dangling pointer** holds the address of memory that has already been **freed** (or that has gone out of scope, in the case of stack memory). Reading or writing through it is undefined behavior — the memory may have been handed out to someone else by the allocator.

## Two Flavors

### Heap dangling pointer
```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
*p = 7;        // dangling — undefined behavior
```

### Stack dangling pointer
```c
int *bad(void) {
    int x = 7;
    return &x;            // x lives on the now-popped stack frame
}

int *p = bad();
*p = 99;                  // dangling — frame is gone
```

## Why It's Dangerous

- The allocator may have **reused** the freed block for a different `malloc`. A write through the dangling pointer **silently corrupts** unrelated data.
- The corruption surfaces *much later*, far from the buggy line, making the bug brutal to debug.
- Read-side use-after-free is a classic **security vulnerability** — attacker-controlled data can end up where the dangling pointer points.

## Common Patterns That Lead Here

- Freeing inside a function while a caller still holds the pointer (broken ownership).
- Storing a pointer to a stack local in a struct that outlives the function.
- Iterating over a list and freeing the current node without first saving `next`.
- Closing a resource handle and then using a still-cached reference to it.

## Defenses

> [!tip] Null after free
> ```c
> free(p);
> p = NULL;
> ```
> A subsequent dereference is now a clean [[Segmentation Fault|segfault]] instead of silent corruption.

- **Ownership discipline** — exactly one place is responsible for freeing.
- **Smart pointers / RAII** (C++ `unique_ptr`/`shared_ptr`).
- **Borrow checking** (Rust) — eliminates dangling pointers at compile time.
- **GC languages** — memory stays live as long as a reference exists; you can't dangle.
- **AddressSanitizer** — flags use-after-free at runtime, including some stack-frame escapes.

## Related Notes

- [[malloc()]], [[free()]] — where the dangling lifecycle starts.
- [[Stack (Runtime)]] — stack-local dangling.
- [[Heap (Runtime)]] — heap dangling.
- [[Memory Leak]] — the opposite mistake.
- [[Double Free]] — a related `free` mishap.
- [[Ch 14 — Interlude - Memory API]].
