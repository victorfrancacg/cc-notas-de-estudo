---
title: Page Cache
tags:
  - concept
  - ostep
  - memory
  - linux
  - cache
type: concept
introduced_in: Ch 23
---

# Page Cache

> [!abstract] Definition
> The **page cache** is the OS's unified RAM cache for all disk-backed memory: file contents read by `read()`/`mmap()`, anonymous pages destined for swap, and block-device metadata. Linux's page cache is **unified** (one cache, one replacement policy). Pages are clean (matches disk) or dirty (modified, needs writeback).

## What Lives in the Page Cache

| Source | Backing |
|---|---|
| File pages from `read()`/`write()` | The file on disk |
| File pages from `mmap()` | Same file |
| Anonymous (heap, stack) pages | Swap (eventually) |
| Block-device metadata (superblocks, inodes, dirent) | The filesystem |

In Linux, **all of these compete for the same RAM** under one replacement policy. There is no separate "buffer cache" anymore (it was unified into the page cache in the 2.4 era).

## Clean vs Dirty

- **Clean**: page in RAM matches what's on disk. Eviction is free — just discard; reuse the frame.
- **Dirty**: page in RAM has been written; disk version is stale. Eviction requires **writeback** (an actual disk write).

Background flusher threads (Linux: per-bdi `flusher` threads; older kernels: `pdflush`) periodically scan dirty pages and write them back, even if no eviction is happening — to limit dirty data outstanding and to amortize writes.

## Replacement: Modified 2Q

Linux's page-cache replacement is a **two-list** approximation of LRU:

```
On first access:  page → INACTIVE list (tail = oldest)
On re-access:     page promoted → ACTIVE list (tail = oldest hot)
On eviction:      take from INACTIVE tail
Periodically:     rebalance active ↔ inactive
```

Why two lists?
- **Single LRU** would let a one-time scan (e.g., `cat` of a 100 GB file) evict everything else.
- **Two lists** keep the hot working set in active; one-time pages flow through inactive without disturbing.

This is **scan-resistant** — important for systems where backup processes, log scans, or large file copies happen alongside latency-critical workloads.

## How a Read Hits the Page Cache

```
read(fd, buf, N):
    if page is in cache:                    // hit
        memcpy from cache page → buf
    else:                                    // miss
        fault → allocate frame → DiskRead
        memcpy from cache page → buf
```

`mmap()` short-circuits the `memcpy`: the program's pointer **is** the cache page. No double-buffering. This is one big reason `mmap()` can outperform `read()` for some workloads.

## Memory Pressure Eviction

When the kernel needs more memory:

1. **First**, drop clean cache pages — free.
2. **Then**, write dirty pages back, then drop them.
3. **Then**, evict anonymous pages to swap (if swap exists and is enabled).
4. **Finally**, OOM kill if all else fails.

The "swappiness" tunable controls how aggressively the kernel reaches step 3 vs step 1/2.

## Why It's a Big Deal

- **File reads are cached for free** — no app needs its own LRU file cache.
- **Multiple processes share** the same file pages — `cat /etc/passwd` from 100 processes uses one physical copy.
- **Writeback batching** — many small writes coalesce into a few large ones.
- **Read-ahead** — sequential access patterns trigger eager prefetching.

This is one of the **biggest performance wins** of a modern OS: the page cache often makes "disk" feel like RAM for hot data.

## Page Cache vs Application Caches

> [!tip] Apps that maintain their own cache often duplicate the page cache
> Databases historically managed their own buffer pool (because the OS cache was naive). Modern DBs (PostgreSQL, especially) often **rely on the OS page cache** instead and only buffer write-ahead-log pages themselves. Some, like MySQL InnoDB, still keep their own cache for fine-grained control.

## Related Notes

- [[Linux Virtual Memory]] — the broader system.
- [[Page Replacement]] — the policy at work.
- [[Clock Algorithm]] — what 2Q approximates and refines.
- [[mmap()]] — the API that exposes the cache directly.
- [[Swap Space]] — where dirty anonymous pages go.
- [[Memory Hierarchy]] — RAM caches disk; the page cache is that caching layer.
- [[Ch 23 — Complete Virtual Memory Systems]].
