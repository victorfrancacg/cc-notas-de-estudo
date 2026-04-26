---
title: Memory Leak
tags:
  - concept
  - ostep
  - memory
  - bug
aliases:
  - Memory Leaks
  - Leak
type: concept
introduced_in: Ch 14
---

# Memory Leak

> [!abstract] Definition
> A **memory leak** is heap memory that the program no longer needs but never returned to the allocator with [[free()]]. Over time, leaked memory accumulates until the process runs out of address space (or the OS kills it for OOM).

## A Minimal Leak

```c
void leak(void) {
    int *p = malloc(sizeof(int));
    // ... use p ...
    return;     // p goes out of scope; no free() called.
                // The bytes on the heap remain allocated but unreachable.
}
```

After enough calls to `leak()`, the heap grows without bound.

## Why It Matters (or Doesn't)

> [!info] Short-lived programs leak harmlessly
> When a process exits, the OS reclaims **all** of its pages — including leaked heap blocks. So a script that allocates, leaks, and exits in 50ms causes no real harm.

> [!warning] Long-running programs leak fatally
> Servers, daemons, the OS kernel itself — never exit, so leaked memory never comes back. Eventually the process exhausts memory and crashes (often the wrong process — the OOM killer can pick a victim other than the leaker).

## Garbage-Collected Languages Aren't Immune

GC frees memory that has **no live references**. If your program *keeps* references — in a global cache, an unbounded queue, an event listener that's never removed — GC sees the memory as live, and you have a leak in disguise.

```python
cache = {}
def query(key):
    if key not in cache:
        cache[key] = expensive(key)   # never evicts → leaks
    return cache[key]
```

## Detection

- **valgrind** — `valgrind --leak-check=yes ./prog` reports unfree'd allocations with stack traces.
- **AddressSanitizer (ASan)** — `-fsanitize=address` plus `LSAN_OPTIONS` for leaks.
- **Heap profilers** — `massif`, jemalloc's profiler, language-specific tools.

## Prevention

- **Match every `malloc` with a `free`** at the same logical layer.
- **Ownership conventions** — make exactly one piece of code responsible for freeing each allocation. (RAII in C++, `unique_ptr`, `defer` in Go, lifetimes in Rust.)
- **Tools** — run leak checkers in CI, not just locally.

## Related Notes

- [[malloc()]], [[free()]] — the API leaks live in.
- [[Heap (Runtime)]] — where the leaked bytes sit.
- [[Dangling Pointer]] — the *opposite* mistake (free too soon vs. free too late).
- [[Double Free]] — another `free`-related bug.
- [[Ch 14 — Interlude - Memory API]].
