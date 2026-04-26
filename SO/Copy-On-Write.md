---
title: Copy-On-Write
tags:
  - concept
  - ostep
  - memory
  - paging
  - optimization
aliases:
  - COW
  - Copy On Write
type: concept
introduced_in: Ch 23
---

# Copy-On-Write (COW)

> [!abstract] Definition
> **Copy-on-write** is the lazy-copy optimization in which two address spaces nominally "have separate copies" of a page but actually **share one read-only physical [[Page Frame|frame]]** until either side **writes** to it. On the first write, the OS traps via [[Page Fault|page fault]], allocates a new frame, copies the contents, and gives the writer a private writable copy. Pioneered by TENEX (1972), refined in [[VAX-VMS|VMS]], and now ubiquitous.

## Why It Exists: `fork()` Without Lunacy

`fork()` semantically duplicates the entire address space of a process. Naively, this means **copying every page** — gigabytes for a large process. In the very common case of `fork()` followed by `exec()`, the child immediately discards everything and loads a new program. **All that copying was for nothing.**

COW makes `fork()` essentially free:

```
Before fork:    Parent → page A (rw)

After fork:     Parent → page A (r--, COW)
                Child  → page A (r--, COW)
                                ↑
                       same physical frame

Child writes:   Parent → page A (r--, COW)
                Child  → page A' (rw)        ← OS allocated, copied, made writable
                                ↑
                       new physical frame
```

If the child immediately `exec()`s, it never writes any of the COW pages — zero copies actually happen.

## Other Uses

### Snapshots

Take a "snapshot" of process or filesystem state by marking everything COW. Reads continue to see the snapshot; writes diverge into private copies. Used by:

- **ZFS / Btrfs** snapshots.
- **VM snapshots** (KVM, VMware).
- **`fork()` for backup** — parent forks, child reads from a frozen snapshot of memory while parent continues.

### Shared Libraries After Modification

`mmap(MAP_PRIVATE, /lib/libc.so)` maps the library COW. All processes share the read-only pages. If a process patches its own code (rare), only that process's copy diverges.

### Container Layered Filesystems

Docker's overlay filesystem uses file-level COW: the read-only base image plus a writable upper layer. Writes copy a file from base to upper before modifying.

## How It Works

1. **Mark both PTEs read-only**, even if the original was writable. Set a "COW" flag in OS bookkeeping (PTE bits or VMA metadata).
2. **On any write**, the [[MMU]] fires a protection fault.
3. **Page-fault handler** sees the COW flag, distinguishes from a real protection violation:
   - Allocate a new physical frame.
   - Copy the original frame's contents.
   - Map the new frame writable in this process's PTE.
   - Other process's PTE is **untouched** (still points at the original).
4. **Reference counting** — when the last reference goes away, the original frame is freed.

## Performance

- **`fork()`** before COW: O(address space size). After COW: O(page-table size). Massive win for large processes.
- **First write to a COW page**: minor fault + frame allocation + memcpy. Adds ~µs per first-write page. Usually invisible compared to other startup costs.
- **Reference counting overhead**: each shared frame needs an atomic increment/decrement. Modern kernels make this very cheap.

## Pitfalls

> [!warning] COW interacts oddly with page-table walkers
> If many processes COW-share a frame, every write fault means walking page tables, allocating, copying, updating. In the worst case (every page COW, every page written), COW is *slower* than eager copy. Real workloads almost never hit this; common-case win is huge.

> [!warning] Real-time applications hate COW
> Unpredictable per-page faults during execution. Latency-sensitive code uses `mlockall()` or pre-faults to avoid surprise.

## History

- **TENEX** (Bobrow et al., 1972) — first COW implementation, for `fork()`.
- **VMS** (1977) — integrated COW with [[Demand Zeroing]] and segmented FIFO replacement.
- **Mach** (1980s) — COW for IPC: send a message, only copy if recipient writes.
- **Linux**, **Windows**, **macOS** — all use COW universally.

## Related Notes

- [[fork()]] — the canonical user.
- [[Demand Zeroing]] — sibling lazy optimization.
- [[Page Fault]] — what triggers the actual copy.
- [[Page Table Entry]] — where the read-only bit and COW flag live.
- [[VAX-VMS]] — early systematic implementation.
- [[Linux Virtual Memory]] — modern implementation.
- [[Ch 23 — Complete Virtual Memory Systems]].
