---
title: Ch 21 — Beyond Physical Memory - Mechanisms
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
type: chapter
book: OSTEP
chapter: 21
pages: 261-272
---

# Ch 21 — Beyond Physical Memory: Mechanisms

> [!abstract] One-sentence summary
> Real address spaces are bigger than physical RAM. The OS extends memory by **swapping** pages to disk: each [[Page Table Entry|PTE]] gets a **present bit**, accesses to non-present pages cause a **[[Page Fault|page fault]]** that traps the OS, and the kernel runs a **page-fault handler** to bring the page in from **[[Swap Space|swap space]]** — possibly evicting another page first ([[Page Replacement|policy]] in Ch 22). All transparent to the running program.

## Crux

> [!example] The Crux: How To Go Beyond Physical Memory
> How can the OS make use of a larger, slower device (disk, SSD) to transparently provide the illusion of a large virtual address space?

## 21.1 Swap Space

A reserved region of disk used to hold pages currently evicted from RAM:

```
Physical Memory                      Swap Space (disk)
+------+------+------+------+        +-----+-----+-----+-----+-----+-----+-----+-----+
|Proc0 |Proc1 |Proc1 |Proc2 |        |Proc0|Proc0|     |Proc1|Proc1|Proc3|Proc2|Proc3|
|VPN 0 |VPN 2 |VPN 3 |VPN 0 |        |VPN 1|VPN 2|     |VPN 0|VPN 1|VPN 0|VPN 1|VPN 1|
+------+------+------+------+        +-----+-----+-----+-----+-----+-----+-----+-----+
PFN 0  PFN 1  PFN 2  PFN 3            blk 0 blk 1 free  blk 3 blk 4 blk 5 blk 6 blk 7
```

The OS records, per evicted page, its **disk address** (block number in swap). Swap space size sets the maximum total memory all processes can use; "infinite" for now.

> [!info] Not the only on-disk source
> Code pages backed by an executable file don't need swap — the OS can re-read them from the binary. Same for `mmap`-ed files. Swap space holds **anonymous** pages (heap, stack) that have no file backing.

## 21.2 The Present Bit

Already in the PTE (Ch 18). When set: page is in RAM. When clear: page is **not present** — could be swapped out, or never mapped at all.

## 21.3 The Page Fault

> [!info] Definition
> A **page fault** is the trap raised when the [[MMU]] reads a PTE with `Present = 0` while servicing a memory access. The hardware delivers control to the OS's **page-fault handler**.

> [!tip] Aside — confusing terminology
> Strictly speaking, "page fault" = page-not-present; an *invalid* access (no PTE at all) is a different exception (segfault). Casually, both get called "page fault". OSTEP author admits the term is overloaded.

## Aside: Why hardware doesn't service page faults

> Reaching disk takes milliseconds; the extra microseconds of a software trap are negligible. The hardware would need to know swap layout, disk drivers, replacement policies — way out of scope. So all systems handle page faults in **software**, even hardware-managed-TLB systems.

## Servicing a Page Fault

```
1. Hardware: TLB miss → walks PTE → Present=0 → trap to OS
2. OS handler:
     a. Read disk address from PTE (OS uses PFN bits as disk-block ptr)
     b. Find a free physical frame (or evict — see 21.4)
     c. Issue disk read → process is BLOCKED
     d. (other processes run during the I/O)
     e. On I/O completion:
          - update PTE: Present=1, PFN=<new frame>
          - mark process READY
3. Hardware: retries instruction → TLB miss → updated PTE → success
```

The faulting instruction restarts; from the program's perspective, only "a slow load".

## 21.4 What If Memory Is Full?

The OS evicts a page to make room — **page-out**. Choosing the victim is the **[[Page Replacement|page-replacement policy]]** ([[Ch 22 — Beyond Physical Memory - Policies|Ch 22]]):

- Wrong choice → program runs at disk speed (10⁴–10⁵× slower than RAM speed). The single most performance-critical decision in the VM system.

## 21.5 Page-Fault Control Flow

### Hardware (during translation)

```c
VPN = (VA & VPN_MASK) >> SHIFT;
(hit, entry) = TLB_Lookup(VPN);
if (hit) {
    // ... normal translation ...
} else {
    PTE = AccessMemory(PTBR + VPN * sizeof(PTE));
    if (!PTE.Valid)             raise(SEGFAULT);
    if (!CanAccess(PTE.Prot))   raise(PROT_FAULT);
    if (PTE.Present)            { TLB_Insert(...); RetryInstruction(); }
    else                        raise(PAGE_FAULT);    // ← swap path
}
```

### Software (OS page-fault handler)

```c
PFN = FindFreePhysicalPage();
if (PFN == -1)                          // no free pages
    PFN = EvictPage();                  // run replacement policy

DiskRead(PTE.DiskAddr, PFN);            // sleep on I/O
PTE.Present = True;
PTE.PFN     = PFN;
RetryInstruction();
```

The retry causes another TLB miss, which now sees `Present=1` and proceeds.

## 21.6 When Replacements Really Occur — Watermarks

Naive design: evict only when memory is fully full. Reality: the OS keeps a small reserve via **high watermark (HW)** and **low watermark (LW)**:

- A background thread (the **swap daemon** / **page daemon**) sleeps until free pages drop below LW.
- Wakes up, evicts pages in batches until free pages reach HW.
- Goes back to sleep.

Benefits of background work:
- **Cluster** evictions — write multiple dirty pages in one disk pass (better seek/rotation behavior).
- **Hide latency** — the foreground fault handler usually finds a free frame already prepared.
- **Idle-time work** — make use of times when CPU and disk would otherwise sit idle.

> [!tip] Tip: Do work in the background
> A recurring OS pattern: defer non-urgent work to a background thread. Examples: dirty-page writeback, journal commits, GC, file cache cleanup. Smooths bursts and exploits otherwise-idle resources.

## 21.7 Summary

> [!example] Crux answered
> The OS pretends memory is bigger than RAM by **swapping pages to disk** and bringing them back on demand. The mechanism: a **present bit** in the PTE, hardware **page-fault traps**, an OS **page-fault handler** that reads from swap and restarts the instruction. Reading a page transparently can take milliseconds — orders of magnitude slower than RAM. Hence the next chapter: **which page to evict** matters enormously.

## Aside: Memory Overlays — the Bad Old Days

Pre-VM, programmers had to manually arrange code/data into memory and out via "overlays" — a tedious, error-prone discipline. Virtual memory replaced overlays with the *transparent* illusion: write your program assuming the address space fits, the OS handles the rest. One of the great quality-of-life wins in OS history.

## Related Notes

- [[Page Fault]] — the trap that drives swapping.
- [[Swap Space]] — the on-disk backing store.
- [[Page-Fault Handler]] — the OS code that services faults.
- [[Demand Paging]] — bring pages in only on access (the dominant policy).
- [[Page Table Entry]] — where the present bit lives.
- [[TLB]] — what the present bit is checked alongside.
- [[Page Replacement]] — Ch 22, deciding what to evict.
- [[Memory Hierarchy]] — RAM-on-top-of-disk is one tier.
- Next: [[Ch 22 — Beyond Physical Memory - Policies]].
- Index: [[MOC - Memory Virtualization]].
