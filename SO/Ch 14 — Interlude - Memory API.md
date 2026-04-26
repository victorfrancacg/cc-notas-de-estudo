---
title: Ch 14 — Interlude - Memory API
tags:
  - ostep
  - chapter
  - interlude
  - part/virtualization
  - memory
type: chapter
book: OSTEP
chapter: 14
pages: 141-151
---

# Ch 14 — Interlude: Memory API

> [!abstract] One-sentence summary
> User-space memory split in two: the **[[Stack (Runtime)|stack]]** (compiler-managed, automatic) and the **[[Heap (Runtime)|heap]]** (programmer-managed via `malloc`/`free`). The chapter is a tour of the heap API and its **classic bugs** — segfaults, leaks, dangling pointers, double frees — plus the OS substrate (`brk`, `mmap`) that powers it.

## Crux

> [!example] The Crux: How To Manage Heap Memory
> What interfaces does the OS / C library provide for allocating and freeing dynamic memory, and what are the common pitfalls programmers face when using them?

## 14.1 Two Types of Memory

| | [[Stack (Runtime)\|Stack]] | [[Heap (Runtime)\|Heap]] |
|---|---|---|
| Allocated by | the compiler (push at call) | the programmer (`malloc`) |
| Freed by | the compiler (pop at return) | the programmer (`free`) |
| Lifetime | function call | as long as you keep the pointer |
| Bug surface | small | large (this chapter) |

The chapter focuses on the heap because it's where things go wrong.

## 14.2 The `malloc()` Call

```c
#include <stdlib.h>
void *malloc(size_t size);
```

- Returns a pointer to a newly allocated `size`-byte block, or `NULL` on failure.
- Use **`sizeof()`** to compute the right size at compile time:

```c
double *d = (double *) malloc(sizeof(double));
int    *a = (int *)    malloc(10 * sizeof(int));    // array of 10 ints
char   *s = (char *)   malloc(strlen(src) + 1);     // +1 for '\0'!
```

> [!warning] `sizeof` on pointers vs arrays
> `sizeof(x)` where `x` is a *pointer* returns the pointer size (4 or 8). For dynamically allocated memory, the C language has **no way** to ask "how big is this `malloc`'d region?" — the size lives only inside the allocator's bookkeeping.

The cast `(double *) malloc(...)` isn't required for correctness — `void *` converts implicitly in C — but communicates intent.

## 14.3 The `free()` Call

```c
void free(void *ptr);
```

You only pass the pointer; the allocator tracked the size internally when it satisfied the matching `malloc()`. See [[Free-Space Management|Ch 17]] for *how* it tracks it (typically a header word just before the returned address).

## 14.4 Common Errors

The whole catalog of heap bugs every C programmer eventually meets:

> [!warning] Forgetting to allocate (a [[Segmentation Fault|segfault]])
> ```c
> char *src = "hello";
> char *dst;                  // uninitialized pointer
> strcpy(dst, src);           // segfault: dst is garbage
> ```

> [!warning] Not allocating enough — [[Buffer Overflow]]
> ```c
> char *dst = malloc(strlen(src));  // forgot the +1 for '\0'
> strcpy(dst, src);                 // writes 1 byte past the end
> ```
> Often appears to "work" — until it doesn't, or until someone exploits it.

> [!warning] [[Uninitialized Memory Read]]
> `malloc` does not zero memory. Reading bytes you never wrote is undefined behavior. Use `calloc` if you want zeros.

> [!warning] [[Memory Leak]]
> `malloc` without a matching `free`. Long-running programs (servers, OS itself) eventually exhaust memory. GC languages can still leak if references are kept alive in caches/closures.

> [!warning] [[Dangling Pointer]]
> Using a pointer **after** the memory it points to was freed. The allocator may have reused that block — your read or write now corrupts unrelated data.

> [!warning] [[Double Free]]
> Calling `free` twice on the same pointer. Corrupts the allocator's free list — symptoms surface much later, far from the buggy call.

> [!warning] Invalid free
> Calling `free` on a pointer not returned by `malloc` (e.g., a stack address, an interior pointer). Undefined behavior; usually a crash.

> [!info] Aside: why short programs "don't leak"
> The OS does memory management at *two levels*. Inside the process, your `malloc`/`free` manage the heap. When the process **exits**, the OS reclaims **all** of its pages — including any leaked heap blocks. So short-lived programs can get away with not freeing. Long-running servers cannot.

## 14.5 Underlying OS Support

`malloc` is **library code**, not a system call. Underneath, it asks the kernel for memory via:

- **[[brk() and sbrk()]]** — old-school: a moveable break pointer at the end of the heap segment.
- **[[mmap()]] (anonymous)** — modern: request a fresh region of pages from the kernel directly. Most large allocations use this path.

> [!tip] Don't call `brk` directly
> Mixing `brk`/`sbrk` calls with `malloc`/`free` will corrupt the allocator's view of the heap. Stick to the library API.

## 14.6 Other Calls

- **`calloc(n, size)`** — like `malloc(n*size)`, but **zeros** the memory. Prevents [[Uninitialized Memory Read|uninitialized-read]] bugs.
- **`realloc(p, size)`** — resize an allocation. May return a *different* pointer (after copying). Always assign the result; never assume in-place resize.

## 14.7 Summary — Tools of the Trade

> [!tip] Don't debug heap bugs by squinting
> Use **valgrind** (`--leak-check=yes`) and **AddressSanitizer** (`-fsanitize=address`). They catch leaks, double frees, dangling reads, and overflows — bugs that compile cleanly and may pass tests by luck.

## Related Notes

- [[Heap (Runtime)]], [[Stack (Runtime)]], [[Address Space]] — what the API operates on.
- [[malloc()]], [[free()]] — the API itself.
- [[brk() and sbrk()]], [[mmap()]] — how the heap actually grows.
- [[Memory Leak]], [[Dangling Pointer]], [[Double Free]], [[Segmentation Fault]], [[Buffer Overflow]], [[Uninitialized Memory Read]] — the bug catalog.
- [[Free-Space Management]] — Ch 17, how the allocator decides what to hand out.
- Next: [[Ch 15 — Mechanism - Address Translation]] — how the hardware maps virtual to physical.
- Index: [[MOC - Memory Virtualization]].
