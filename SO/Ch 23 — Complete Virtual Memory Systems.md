---
title: Ch 23 — Complete Virtual Memory Systems
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
  - case-study
type: chapter
book: OSTEP
chapter: 23
pages: 291-309
---

# Ch 23 — Complete Virtual Memory Systems

> [!abstract] One-sentence summary
> Two case studies of full VM systems: **VAX/VMS** (1970s, designed by Dave Cutler — later of Windows NT) — a clean classical design that emulated reference bits, segmented FIFO, demand zeroing, [[Copy-On-Write|copy-on-write]]. And **Linux** — the modern workhorse with hardware-walked multi-level page tables, [[Huge Pages|huge pages]], unified [[Page Cache|page cache]], and a security stack (NX, [[ASLR]], KASLR, KPTI) born of decades of attacks ([[Buffer Overflow]], ROP, Meltdown/Spectre).

## Crux

> [!example] The Crux: How To Build A Complete VM System
> What features are needed to realize a complete virtual memory system? How do they improve performance, increase security, or otherwise improve the system?

## 23.1 VAX/VMS Virtual Memory

### Hardware

- **32-bit VA**, **512-byte pages**.
- VA splits: 2 bits for segment, 21 bits for VPN, 9 bits for offset.
- Three segments: **P0** (process code/heap, lower half), **P1** (stack, lower half growing down), **S** (system/kernel, upper half — shared across processes).
- Hybrid paging+segmentation — base+bound registers point at per-segment **page tables** (rather than at the segment data directly).

> [!warning] Tiny page size hurts
> 512 B pages → linear page tables explode. VMS partly mitigated by per-segment tables (sparse address spaces stay sparse) and by **putting kernel page tables in kernel virtual memory** (so they themselves can be swapped — a wild move).

### Real Address Space

```
0          ┌────────────┐  Page 0: INVALID — null-pointer detection!
           │ User Code  │
           │ User Heap  │  P0 — process space
2^30       ├────────────┤
           │   ...      │  (gap)
           │ User Stack │  P1 — process space (grows down)
2^31       ├────────────┤
           │ Trap Tables│
           │ Kernel Data│
           │ Kernel Code│  S — system space, shared across processes
           │ Kernel Heap│
2^32       └────────────┘
```

> [!tip] Page 0 marked invalid → null-pointer dereference is a clean trap
> Many later systems adopted this: dereferencing `NULL` (= virtual address 0) faults instead of silently corrupting. Modern Linux does the same.

### No Reference Bit — Emulated

VAX hardware lacked a [[Reference Bit|use bit]]. VMS emulated it via **protection bits** (Babaoglu & Joy, 1981):

1. Mark a page **inaccessible** in the PTE.
2. On access, the hardware traps to OS.
3. OS notes "page was referenced", restores normal protection, returns.
4. Periodically re-mark pages inaccessible to test again.

Effective; a clever software workaround for a hardware shortcoming.

### Segmented FIFO + Second-Chance Lists

VMS used [[Page Replacement|page replacement]] with:

- **Per-process resident set size (RSS)** — fixed quota of pages each process can keep in RAM.
- When a process exceeds its RSS, the **oldest page** in its FIFO is evicted *from the process* (local replacement).
- Evicted pages go onto **global second-chance lists**:
  - **Clean** evictees → "clean-page free list" (instantly reclaimable; just discard if reused).
  - **Dirty** evictees → "dirty-page list" (must be written to disk).
- If the original process re-references a page on those lists, it reclaims the page (no disk I/O — a "soft" minor fault).
- **Clustering**: dirty pages are grouped before being written, amortizing disk seeks.

The result: a FIFO-with-second-chance approximation that performs close to LRU at low cost.

### Demand Zeroing

Naive: when a process asks for a new heap page, the OS finds a frame, zeroes it (security!), maps it. Slow if the process never touches the page.

VMS: mark the new page **inaccessible** but flagged "demand-zero". On first access, trap to OS, allocate frame, zero, map, return. **Pay the cost only when needed.** See [[Demand Zeroing]].

### [[Copy-On-Write]]

Originally TENEX (1972). When the OS would copy a page from one address space to another, instead **map it read-only into both** and copy lazily on the first write. Critical for `fork()` — a fork followed by `exec()` would otherwise copy the entire address space for nothing.

## 23.2 The Linux Virtual Memory System

### Address Space (32-bit Linux on x86)

```
0x00000000  ┌────────────┐  Page 0 invalid
            │ User Code  │
            │ User Heap  │
            │   ...      │  User space (3 GB)
            │ User Stack │
0xC0000000  ├────────────┤
            │ Kernel     │  Kernel space (1 GB), shared across all processes
            │ (Logical)  │
            │ Kernel     │
            │ (Virtual)  │
0xFFFFFFFF  └────────────┘
```

64-bit Linux has the same structure with much larger windows.

### Kernel Logical vs Kernel Virtual Memory

- **Kernel logical addresses** — direct-mapped to physical RAM (`virtual = physical + 0xC0000000`). Fast translation, contiguous in physical too, suitable for **DMA**. Allocated via `kmalloc`. **Cannot be swapped**.
- **Kernel virtual addresses** — go through page tables; need not be physically contiguous. Allocated via `vmalloc`. Useful for large buffers where physical contiguity would be hard.

### Page Table Structure

- x86 hardware-managed 4-level page table (32-bit had 2-level + PAE; 64-bit uses 4-level).
- OS sets up tables, loads `CR3`; hardware walks on TLB miss.
- Per-process tables; **kernel mappings duplicated** in each process's upper half so syscalls don't switch tables.

### [[Huge Pages]]

x86 supports 4 KB, **2 MB**, **1 GB** pages. Linux exposes:

- **Explicit huge pages** — apps request via `mmap(MAP_HUGETLB)` or `shmget(SHM_HUGETLB)`. Used by databases, JVMs, ML frameworks for large structures.
- **Transparent huge pages (THP)** — kernel automatically promotes 4 KB pages to 2 MB when contiguous regions are available. Best of both worlds; can also cause latency spikes during compaction.

Benefits: fewer [[TLB]] misses (1 entry covers 2 MB instead of 4 KB), fewer page-table entries.

> [!warning] Huge pages aren't free
> [[Internal Fragmentation]] — a 2 MB page allocated for sparse data wastes RAM. Sparse-but-large workloads suffer.

### The [[Page Cache]]

Linux's unified cache for **all** disk-backed memory:

- File pages from `read()`/`mmap()`.
- Anonymous pages destined for swap.
- Block-device metadata.

All live in the same hash-keyed structure, with a single replacement policy. Pages are **clean** (matches disk) or **dirty** (modified). A background thread (`pdflush` historically; `flusher` threads today) writes dirty pages back periodically.

Replacement: a **modified 2Q** (Johnson & Shasha, 1994) — **active list** + **inactive list**:

- New pages → inactive list (hits the LRU end first).
- Re-referenced → promoted to active list.
- Eviction always from the inactive tail.
- Periodically rebalance.

Scan-resistant: a one-time large file read flows through the inactive list without disturbing the active list.

### Memory Mapping Is Everywhere

> [!info] `pmap <pid>` reveals
> ```
> 0000000000400000   372K r-x--  tcsh             (executable code)
> 00000000019d5000  1780K rw---  [anon]           (heap)
> 00007f4e7cf06000  1792K r-x--  libc-2.23.so     (library code)
> 00007f4e7d2d0000    36K r-x--  libcrypt-2.23.so
> 00007f4e7d508000   148K r-x--  libtinfo.so.5.9
> 00007f4e7d731000   152K r-x--  ld-2.23.so       (dynamic linker)
> 00007f4e7d932000    16K rw---  [stack]
> ```
> Every regular Linux process is a patchwork of memory-mapped files (executable, libraries) plus anonymous regions (heap, stack). [[mmap()]] is the universal mechanism.

### Security Stack

VMS lived in a gentler era; modern Linux deals with adversaries.

**[[Buffer Overflow]] → control-flow hijack**
The classic stack-smashing attack overwrites saved return addresses to redirect execution.

**NX bit (No-Execute, a.k.a. XD on Intel)**
Per-page flag in the PTE. Pages marked non-executable trap on instruction fetch. Stops shellcode injected onto the stack from running.

**Return-Oriented Programming (ROP)**
Defenders add NX → attackers respond by **chaining existing code fragments** ("gadgets") via the stack. Doesn't need executable injection, just a corrupted stack pointing at well-chosen gadget addresses.

**[[ASLR|Address Space Layout Randomization]]**
Each process load randomizes the base addresses of code, libraries, stack, heap. Attackers can't hard-code target addresses. **KASLR** does the same for the kernel.

**Meltdown and Spectre (2018)**
Speculative execution leaves traces in cache state, even when the CPU rolls back the speculation. Attackers can read **other processes'** (or even the kernel's) memory by training the CPU to speculate down attacker-chosen paths and then measuring cache timings.

**KPTI (Kernel Page-Table Isolation)**
The Meltdown countermeasure. Instead of mapping all of kernel memory into every user address space, give the kernel its own page table. Switching to/from the kernel now involves a page-table swap (slower) but prevents the speculative kernel-memory leak. **A modern reversal of the "kernel-in-every-process" trick** that VAX/VMS pioneered.

## 23.3 Summary

> [!example] Crux answered
> Real VM systems combine the mechanisms of Ch 13–22 with practical engineering: [[Demand Zeroing]] and [[Copy-On-Write]] for laziness, segmented FIFO with second-chance lists for cheap LRU approximation, [[Huge Pages]] for TLB coverage, unified [[Page Cache|page caches]] for memory-disk sharing, and security primitives ([[ASLR]], NX, KPTI) born of attacker pressure. The 50-year-old VMS ideas live on; modern systems add the security layer the original era didn't need.

## Aside: Be Lazy

> [!tip] Laziness is an OS virtue
> [[Demand Paging]], demand zeroing, [[Copy-On-Write]] — all rely on **deferring work** until provably needed. Sometimes the work never happens (the page is freed, the file is deleted). When it does happen, you've at least improved responsiveness. The same principle drives lazy file I/O, lazy GC, lazy compilation.

## Aside: Incrementalism

> [!tip] Linux's huge-page rollout is exemplary
> Linux didn't ship transparent huge pages immediately. First, opt-in (`mmap(MAP_HUGETLB)`) for the few apps that demanded them. Years of experience exposed costs (compaction stalls, internal fragmentation). Then THP made it automatic. **Slow, thoughtful incrementalism** beat revolution.

## Related Notes

- [[VAX-VMS]] — case-study atomic note.
- [[Linux Virtual Memory]] — case-study atomic note.
- [[Copy-On-Write]] — used by `fork()`, modern shared memory.
- [[Demand Zeroing]] — VMS innovation, now universal.
- [[Page Cache]] — Linux's unified file-and-swap cache.
- [[Huge Pages]] — TLB-coverage win.
- [[ASLR]] — security baseline.
- [[Buffer Overflow]] — the attack class that started it all.
- [[Meltdown and Spectre]] — the speculative-execution upheaval.
- [[fork()]] — relies on COW.
- [[mmap()]] — the universal Linux mapping primitive.
- Next: [[Ch 24 — Summary Dialogue on Memory Virtualization]] — closing dialogue.
- Index: [[MOC - Memory Virtualization]].
