---
title: mmap()
tags:
  - concept
  - ostep
  - memory
  - syscall
aliases:
  - mmap
  - Memory Mapping
type: concept
introduced_in: Ch 14
---

# `mmap()`

> [!abstract] Definition
> `mmap()` maps a region of the process's [[Address Space]] to either a **file** (file-backed mapping) or **fresh pages of zero-filled memory** (anonymous mapping). It is the modern, flexible alternative to [[brk() and sbrk()|brk]] for obtaining heap memory and the universal mechanism for memory-mapped I/O.

## API

```c
#include <sys/mman.h>

void *mmap(void *addr,
           size_t length,
           int prot,           // PROT_READ | PROT_WRITE | PROT_EXEC
           int flags,          // MAP_PRIVATE | MAP_SHARED | MAP_ANONYMOUS | ...
           int fd,             // file descriptor (or -1 for anonymous)
           off_t offset);

int munmap(void *addr, size_t length);
```

## Two Big Use Cases

### 1. Anonymous mapping — fresh memory

```c
void *p = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
               MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
```

- Returns 4 KB of zero-filled, private memory.
- Modern allocators ([[malloc()|malloc]], jemalloc) use this for large allocations and per-thread arenas — `munmap` can release them back to the OS at arbitrary times, unlike [[brk() and sbrk()|brk]].

### 2. File-backed mapping — memory-mapped I/O

```c
int fd = open("data.bin", O_RDONLY);
void *p = mmap(NULL, sb.st_size, PROT_READ,
               MAP_PRIVATE, fd, 0);
// access file contents as if they were an array in memory:
char first_byte = ((char *)p)[0];
```

- Reads/writes through `p` are translated into reads/writes on `data.bin` by the kernel.
- Pages are demand-paged — only loaded when first touched.
- The kernel's **page cache** is shared between `read()`/`write()` and `mmap` — no double buffering.

## Flags Cheat Sheet

| Flag | Meaning |
|---|---|
| `MAP_PRIVATE` | Writes are private (copy-on-write); don't propagate to file |
| `MAP_SHARED` | Writes propagate to the file and to other mappers |
| `MAP_ANONYMOUS` | No file backing; pages start zero-filled |
| `MAP_FIXED` | Use exactly `addr`; refuse to remap. Dangerous |
| `MAP_HUGETLB` | Use huge pages (2 MB / 1 GB) — fewer [[TLB]] misses |

## Why Modern Allocators Love `mmap`

- **Released memory actually returns to the OS** — `munmap` un-maps; the kernel can reuse those pages elsewhere.
- **Independent regions** — many `mmap` arenas can coexist; freeing one doesn't depend on what's "above" it (unlike `brk`).
- **Per-thread arenas** — each thread can `mmap` its own arena, dodging lock contention.

## Memory-Mapped I/O Trade-Offs

> [!tip] When `mmap` beats `read`
> - Random access into a large file (no need to issue many `read` calls).
> - Sharing a file between processes via `MAP_SHARED`.
> - Treating the file as if it were a giant `char *`.

> [!warning] When `mmap` is worse
> - Sequential streaming — `read` is simpler and may be faster.
> - Files that may be truncated under you — `SIGBUS` on access.
> - Many small files — per-mapping overhead adds up.

## Related Notes

- [[Address Space]] — what `mmap` modifies.
- [[malloc()]], [[free()]] — built atop `mmap` for large allocations.
- [[brk() and sbrk()]] — the older, less flexible alternative.
- [[Heap (Runtime)]] — modern heaps are largely `mmap`-backed.
- [[Code Segment]] — loader uses `mmap` to map a binary's `.text`.
- [[Page Fault]] — how demand paging fills in `mmap`'d pages on first touch.
- [[Ch 14 — Interlude - Memory API]].
