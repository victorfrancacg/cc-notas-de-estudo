---
title: calloc()
tags:
  - concept
  - ostep
  - memory
  - api
  - libc
aliases:
  - calloc
type: concept
introduced_in: Ch 14
---

# calloc()

> [!abstract] Definition
> `calloc(num, size)` allocates `num * size` bytes on the [[Heap (Runtime)|heap]] **and zero-initializes them**, returning a pointer (or `NULL`). Equivalent to `malloc(num * size) + memset(p, 0, num * size)` but typically faster because the kernel can hand back demand-zeroed pages.

## Signature

```c
#include <stdlib.h>

void *calloc(size_t num, size_t size);
```

```c
int *zeroed_array = calloc(100, sizeof(int));   // 100 ints, all 0
struct s *q = calloc(1, sizeof(*q));            // zero-init a struct
```

## Why Not Just `malloc + memset`?

- Same effect, but `calloc` can be **cheaper**: when the OS gives the allocator fresh pages via [[mmap()|anonymous `mmap()`]], those pages are already zero (the kernel zero-fills for security). `calloc` knows this and skips the redundant `memset`.
- Reused (already-touched) pages still get explicitly zeroed.

## When To Use `calloc`

- Whenever you'd otherwise zero the memory yourself.
- Any structure with pointer or fd fields where zero ≡ "no allocation, no file" — prevents [[Uninitialized Memory Read|uninitialized-read]] bugs from surfacing later.

## Watch Out: Multiplication Overflow

```c
calloc(huge_n, huge_size);   // n * size may overflow size_t
```

`calloc` is required to detect this and return `NULL`; portable code shouldn't rely on it but recent C standards require it.

## Related Notes

- [[malloc()]] — sibling that does *not* zero.
- [[realloc()]] — sibling for resizing.
- [[free()]] — for releasing.
- [[Uninitialized Memory Read]] — the bug `calloc` prevents.
- [[Ch 14 — Interlude - Memory API]].
