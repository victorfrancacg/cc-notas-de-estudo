---
title: Demand Zeroing
tags:
  - concept
  - ostep
  - memory
  - paging
  - optimization
type: concept
introduced_in: Ch 23
---

# Demand Zeroing

> [!abstract] Definition
> **Demand zeroing** is the lazy-allocation technique where the OS, when asked to provide a fresh anonymous [[Page|page]] (e.g., a new heap page), does **not** immediately allocate a [[Page Frame|frame]] and zero it. Instead, it sets up an invalid PTE marked "demand-zero". On the first access, a [[Page Fault|page fault]] triggers; the [[Page-Fault Handler|handler]] then allocates and zeros a frame. **Zero work is done if the page is never touched.**

## The Eager Alternative (and Why It's Wasteful)

Naive `malloc(1 GB)` followed by usage of only the first 16 KB:

```
Eager:
  - Allocate 256K frames.
  - Zero each (security: don't leak previous owner's data).
  - Map all 256K PTEs.
  → 1 GB of physical memory consumed.
  → Heavy memory bandwidth burned on zeroing.
  → Process uses 16 KB; the other 1023.984 GB is wasted.
```

Programs routinely allocate large buffers and use only a fraction. Eager allocation is a disaster.

## Demand Zeroing: The Lazy Win

```
Lazy:
  - Mark VMA as anonymous, demand-zero.
  - PTEs all invalid (or pointing at /dev/zero on Linux).
  - On first access to each page:
       trap → handler → allocate frame → zero → map.
  → Only the *touched* pages cost physical RAM.
  → Allocation is essentially free; zero work upfront.
```

`malloc(1 GB)` returns instantly; you only pay for what you use.

## Implementation Sketch

```c
// Allocation
mmap(NULL, size, PROT_READ|PROT_WRITE, MAP_ANONYMOUS|MAP_PRIVATE, -1, 0);
// → record VMA; no PTEs allocated yet.

// First access (page fault):
on page fault to VPN V in anonymous region:
    if PTE invalid:
        frame = alloc_zeroed_frame();
        map_writable(V, frame);
        retry instruction;
```

`alloc_zeroed_frame()` itself can have its own optimizations:

- **Pre-zeroed page pool** — a kernel idle thread keeps a small reserve of zeroed frames. The fault path takes one without zeroing.
- **`/dev/zero` mapping** — file-backed COW from a zero-source. First write triggers COW, allocating and "copying" zero (fast).

## Security: Why Zeroing Matters

A frame freed by Process A might contain credit card numbers. Reusing that frame for Process B without zeroing leaks A's data. Zeroing is a **security requirement**, not just hygiene.

But the cost can be deferred — **demand-zero defers the cost to the moment of need**, not allocation time.

## Where It Lives

Every modern OS uses demand zeroing for anonymous memory:

- Linux: every `mmap(MAP_ANONYMOUS)` page is demand-zero.
- Windows: same.
- macOS: same (Mach roots).

The user-facing impact: `malloc(N)` runs in O(1) regardless of N — the work is amortized over the actual writes.

## Combination with Other Lazy Tricks

- **[[Copy-On-Write]]** — share a single zero-page among all uninitialized anonymous mappings (Linux uses one **system-wide read-only zero page**). On first write, COW kicks in: allocate a private frame, copy-zero, mark writable.
- **`MAP_POPULATE`** — explicit eager allocation when you really do want all pages mapped now (e.g., real-time apps).
- **`mlock()`** — pin pages to RAM, forcing immediate allocation and disabling demand-paging.

## Related Notes

- [[Copy-On-Write]] — sibling lazy mechanism.
- [[Demand Paging]] — broader idea of "do work only when needed".
- [[Page Fault]] — what triggers the actual allocation.
- [[mmap()]] — the API anonymous mappings come through.
- [[VAX-VMS]] — where the technique was systematically deployed.
- [[Linux Virtual Memory]] — modern implementation.
- [[Ch 23 — Complete Virtual Memory Systems]].
