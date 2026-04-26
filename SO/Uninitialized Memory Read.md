---
title: Uninitialized Memory Read
tags:
  - concept
  - ostep
  - memory
  - bug
aliases:
  - Uninitialized Read
type: concept
introduced_in: Ch 14
---

# Uninitialized Memory Read

> [!abstract] Definition
> Reading from memory that was allocated but never written produces an **undefined value** — whatever bytes happened to be there. In C, this includes `malloc`'d memory, freshly declared local variables, and certain struct fields. The bug is silent, non-deterministic, and a frequent source of "works on my machine".

## Where It Comes From

```c
int *p = malloc(sizeof(int));
printf("%d\n", *p);     // undefined: malloc does not zero memory
```

```c
int x;
if (x > 0) { ... }      // undefined: x is uninitialized
```

```c
struct s { int a; int b; };
struct s *q = malloc(sizeof(*q));
// q->a and q->b are both garbage until written.
```

## Why It's Insidious

- The garbage value may **happen to be zero** if the OS just gave the process a fresh page (which `mmap` zero-fills for security). The program seems to work — until the bytes are reused and contain something else.
- Tests pass; production fails.
- Compiler optimizations may **assume** undefined behavior doesn't happen — and emit code that does surprising things if it does.

## Defenses

> [!tip] Use [[calloc()]] when you want zeros
> `calloc(n, size)` allocates and **zeros** memory in one step. Slightly slower than `malloc`, but the cost is paid only on the first touch (the kernel demand-zeros pages anyway).

- **Initialize at declaration** — `int x = 0;` not `int x;`.
- **Initialize structs with `{0}`** or designated initializers.
- **`memset(p, 0, n)`** after `malloc` if you need zero-init.
- **Compiler warnings** — `-Wuninitialized`, `-Wmaybe-uninitialized` catch many cases.
- **Valgrind's memcheck** — catches reads of uninitialized bytes at runtime.
- **MSan (MemorySanitizer)** — Clang/LLVM tool; tracks uninitialized propagation across the program.

## Why Not Always Zero `malloc`?

Zeroing has a cost (memory bandwidth). Allocators leave the choice to the programmer:

- `malloc` — fast; you fill in what you need.
- `calloc` — slower (or not, on first-touch demand-zeroed pages); safe by default.

## Related Notes

- [[malloc()]] — does *not* zero.
- [[calloc()]] — does zero.
- [[Heap (Runtime)]] — where the uninitialized bytes live.
- [[Ch 14 — Interlude - Memory API]].
