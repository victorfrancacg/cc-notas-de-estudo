---
title: Double Free
tags:
  - concept
  - ostep
  - memory
  - bug
aliases:
  - Double-Free
type: concept
introduced_in: Ch 14
---

# Double Free

> [!abstract] Definition
> A **double free** is calling [[free()]] twice on the same pointer (without an intervening [[malloc()]] returning that same pointer). The allocator's internal data structures get corrupted; the bug usually surfaces as a crash or memory corruption far from the offending call.

## Minimal Example

```c
int *p = malloc(sizeof(int));
free(p);
free(p);            // double free — undefined behavior
```

## Why It's Toxic

The allocator records that a block is free by linking it into a **free list**. A second `free(p)` may:

- Link the block onto the list **twice** — a future `malloc` returns the same block to two callers, who then write over each other.
- Use a freed-block header word as a free-list pointer, even though that memory was already handed back to a *different* `malloc` caller — leading to wild writes.
- Trigger an explicit allocator check (glibc detects some double-frees and aborts: `*** glibc detected *** double free or corruption ***`).

The crash, if any, surfaces **at the next `malloc` or `free`** — not at the buggy line.

## Common Causes

- Two cleanup paths both freeing the same resource.
- A destructor running twice (e.g., a copy that owns the same pointer as the original).
- Aliasing — two pointers that name the same allocation, both freed.
- Re-running a cleanup function after a partial init failure.

## Defenses

> [!tip] Null after free
> ```c
> free(p);
> p = NULL;     // future free(p) is now a no-op (free(NULL) is safe)
> ```

- **Single ownership** — exactly one piece of code is responsible for freeing.
- **Smart pointers** (C++ `unique_ptr`) — moves transfer ownership; cannot be freed twice.
- **Linear / affine types** (Rust) — the borrow checker prevents this entirely.
- **AddressSanitizer / glibc's MALLOC_CHECK_** — runtime detection.

## Related Notes

- [[free()]] — the call being misused.
- [[malloc()]] — its partner.
- [[Memory Leak]], [[Dangling Pointer]] — sibling bugs.
- [[Free-Space Management]] — Ch 17, how free lists work.
- [[Ch 14 — Interlude - Memory API]].
