---
title: Page-Fault Handler
tags:
  - concept
  - ostep
  - memory
  - kernel
aliases:
  - Page Fault Handler
  - PF Handler
type: concept
introduced_in: Ch 21
---

# Page-Fault Handler

> [!abstract] Definition
> The **page-fault handler** is the OS-kernel code that runs when the hardware raises a [[Page Fault|page fault]]. Its job: figure out *why* the fault happened, fetch any needed page from disk (running [[Page Replacement|replacement]] if memory is full), update the [[Page Table Entry|PTE]] to point at the new physical frame, and return control so the faulting instruction retries.

## The Algorithm

```c
void page_fault_handler(virtual_address VA) {
    PTE *pte = walk_page_table(current_process, VA);

    if (pte == NULL || !pte->Valid) {
        // unmapped — this is a segfault, not a swap-in
        deliver_signal(SIGSEGV);
        return;
    }

    if (!can_access(pte->Prot, faulting_op)) {
        deliver_signal(SIGSEGV);  // protection violation
        return;
    }

    // Genuine swap-in path
    PFN frame = find_free_physical_page();
    if (frame == -1)
        frame = evict_page();              // run replacement policy

    if (pte->BackingStore == SWAP)
        DiskRead(swap_addr(pte), frame);   // sleep on I/O
    else if (pte->BackingStore == FILE)
        DiskRead(file_addr(pte), frame);
    else
        zero_page(frame);                   // anonymous demand-zero

    pte->Present = 1;
    pte->PFN     = frame;
    // (do NOT explicitly update TLB — retry will re-walk and refill)

    return;   // hardware retries the faulting instruction
}
```

Real Linux's `do_page_fault()` is far more elaborate (handles COW, NUMA migration, transparent huge pages, KSM, etc.) but follows this skeleton.

## Why Software, Not Hardware

The handler must:

- Speak to the disk driver.
- Run the replacement policy.
- Allocate frames from a complex free-list structure.
- Update kernel data structures.

None of that is in scope for the [[MMU]]'s circuitry. Plus, page-fault latency is dominated by **disk I/O** (milliseconds); the few microseconds of trap+handler overhead are noise. So even systems with hardware-managed TLBs handle page faults in software.

## Process State During a Fault

While the handler waits on disk:
- Faulting process is **Blocked** (see [[Process States]]).
- The CPU runs other ready processes.
- I/O completion wakes the faulting process; it returns to **Ready**.
- Eventually scheduled; instruction retries.

That overlap is one of the original justifications for [[Multiprogramming]] — overlap I/O of one process with computation of another.

## Common Fault Scenarios

| Scenario | What handler does |
|---|---|
| First touch of `mmap()`-ed memory | Allocate frame, zero it (anonymous) or read from file |
| First write to a fork'd page | COW: allocate new frame, copy contents, mark writable |
| Touched a swapped-out anon page | Read from swap into a frame |
| Touched a discarded code page | Re-read from binary |
| Wild pointer / NULL deref | Deliver SIGSEGV, kill |
| Write to read-only page | Deliver SIGSEGV (or, if MAP_SHARED file, allow) |

## Related Notes

- [[Page Fault]] — the trap event.
- [[Swap Space]] — where most "missing" pages are.
- [[Page Replacement]] — the eviction step.
- [[Demand Paging]] — driven by the handler.
- [[Trap]], [[Trap Table]] — how the fault enters the kernel.
- [[Process States]] — Blocked while waiting on I/O.
- [[fork()]] — its COW mechanism leverages page faults.
- [[Ch 21 — Beyond Physical Memory - Mechanisms]].
