---
title: Heap (Runtime)
tags:
  - concept
  - ostep
  - memory
  - address-space
aliases:
  - Heap
  - Runtime Heap
  - Process Heap
type: concept
introduced_in: Ch 13
detailed_in: Ch 14
---

# Heap (Runtime)

> [!abstract] Definition
> The **heap** is the region of a process's [[Address Space]] used for **dynamically allocated** memory — memory whose lifetime is decided at run time rather than at compile time. In C it is reached via [[malloc()]] / [[free()]]; in C++/Java via `new` / `delete` (or the GC). Conventionally placed just above the [[Code Segment|code]] and **grows upward** toward the [[Stack (Runtime)|stack]].

## Position in the Address Space

```
   0KB  ┌──────────────────┐
        │ Code             │  static
   1KB  ├──────────────────┤
        │ Heap             │  grows ↓
   2KB  │     ↓            │
        │  (free)          │
        │     ↑            │
  15KB  │ Stack            │  grows ↑
  16KB  └──────────────────┘
```

The heap and [[Stack (Runtime)|stack]] sit at opposite ends so each can grow without colliding until they meet in the middle.

## Lifetime Management

| Heap | Stack |
|---|---|
| Explicit (`malloc`/`free`) or GC | Implicit — function entry/exit |
| Lives until you free it | Lives until the function returns |
| Hard to use correctly | Easy and free |
| Fragmenting | Compact |

> [!warning] Common heap bugs
> - **[[Memory Leak]]** — allocated but never freed. Memory usage grows indefinitely.
> - **[[Dangling Pointer]]** — pointer used after the memory was freed.
> - **[[Double Free]]** — `free()` called twice on the same pointer; corrupts allocator state.
> - **[[Uninitialized Memory Read]]** — reading freshly `malloc`'d bytes that hold garbage.
> - **Heap buffer overflow** — writing past the end of a `malloc`'d region.

## How the Heap Grows

User-level allocators sit on top of OS calls that resize the heap region:

- **[[brk() and sbrk()]]** — classic UNIX way: a single moveable "break" pointer marks the end of the heap segment.
- **[[mmap()]]** — modern allocators (e.g., glibc `ptmalloc`, jemalloc, tcmalloc) use anonymous `mmap()` for large allocations and arenas.

The OS gives the allocator big chunks; the allocator carves them into small `malloc`-sized pieces (this is [[Free-Space Management]], Ch 17).

## Heap vs Stack — When to Use Which

> [!tip] Rule of thumb
> Use the [[Stack (Runtime)|stack]] for things whose lifetime matches the enclosing function (locals, small structs). Use the heap when the lifetime must outlast the function's return *or* when the size is too large for the stack (which is finite, often ~8MB).

## Heap and Memory Virtualization

The heap is **virtual** like everything else in the address space. When `malloc(1024)` returns a pointer, the OS may not have actually allocated 1024 physical bytes yet — it may **lazily** map physical pages on first access (demand paging, Ch 21). This is invisible to the programmer thanks to [[Memory Virtualization|memory-virtualization transparency]].

## Related Notes

- [[Address Space]] — the heap is one of its three canonical regions.
- [[Stack (Runtime)]] — the heap's neighbor (and opposite).
- [[Code Segment]] — static instructions; doesn't grow.
- [[malloc()]], [[free()]] — the user-facing API.
- [[brk() and sbrk()]], [[mmap()]] — the kernel boundary.
- [[Free-Space Management]] — how the allocator picks free blocks (Ch 17).
- [[Memory Leak]], [[Dangling Pointer]], [[Double Free]] — classic bugs.
