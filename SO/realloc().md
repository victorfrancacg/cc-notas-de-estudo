---
title: realloc()
tags:
  - concept
  - ostep
  - memory
  - api
  - libc
aliases:
  - realloc
type: concept
introduced_in: Ch 14
---

# realloc()

> [!abstract] Definition
> `realloc(ptr, new_size)` resizes the heap allocation pointed to by `ptr` to `new_size` bytes. It may **return a different pointer** (the allocator copied your data to a new block); always reassign. Returns `NULL` on failure — and **does not free the original block** in that case.

## Signature

```c
#include <stdlib.h>

void *realloc(void *ptr, size_t new_size);
```

## Idiom

```c
int *p = malloc(n * sizeof(int));
// ... outgrew it ...
int *tmp = realloc(p, 2 * n * sizeof(int));
if (tmp == NULL) {
    free(p);                    // realloc didn't free p on failure
    // handle error
} else {
    p = tmp;                    // commit only on success
}
```

> [!warning] Don't write `p = realloc(p, ...)`
> If `realloc` fails it returns `NULL`. Assigning that overwrite leaks the original block — you can't `free` it because you've lost the pointer.

## Edge Cases

- `realloc(NULL, n)` ≡ `malloc(n)`.
- `realloc(p, 0)` ≡ `free(p)` (implementation-defined since C23; previously varied).
- Shrinking: typically returns the same pointer with the trailing bytes released to the free list.
- Growing: may extend in place, or allocate a new block and `memcpy` the contents.

## Performance Hint — Doubling

When growing a dynamically sized buffer (e.g., a vector), **double** the capacity instead of incrementing by one. This makes total work `O(n)` amortized; otherwise it's `O(n²)`.

```c
if (used == capacity) {
    capacity *= 2;
    arr = realloc(arr, capacity * sizeof(*arr));
}
```

## Related Notes

- [[malloc()]] — initial allocation.
- [[free()]] — explicit release.
- [[calloc()]] — zero-initializing allocator.
- [[Heap (Runtime)]] — where reallocation happens.
- [[Free-Space Management]] — Ch 17, why in-place grow is sometimes possible.
- [[Ch 14 — Interlude - Memory API]].
