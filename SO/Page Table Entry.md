---
title: Page Table Entry
tags:
  - concept
  - ostep
  - memory
  - paging
aliases:
  - PTE
type: concept
introduced_in: Ch 18
---

# Page Table Entry (PTE)

> [!abstract] Definition
> A **page table entry (PTE)** is one row of a [[Page Table]]: it records the [[Page Frame|physical frame]] backing a [[Page|virtual page]] and a handful of permission/state bits. The [[MMU]] reads a PTE on every translation (or every [[TLB]] miss) to decide whether and how to access memory.

## Anatomy

A typical x86 PTE:

```
31           12 11   8 7 6 5 4 3 2 1 0
+-------------+-----+-+-+-+-+-+-+-+-+-+
|     PFN     | ... |G|D|A|PCD|PWT|U/S|R/W|P|
+-------------+-----+-+-+-+-+-+-+-+-+-+
```

| Bit / field | Meaning |
|---|---|
| **PFN** | physical frame number — the actual mapping |
| **P (Present)** | resident in RAM? If not → page fault, OS may swap in |
| **R/W** | read-only vs read-write |
| **U/S** | user vs supervisor (kernel-only) |
| **A (Accessed)** | set by hardware when touched; cleared by OS for [[Page Replacement\|replacement]] policy |
| **D (Dirty)** | set by hardware on write; OS must flush dirty pages on eviction |
| **PWT / PCD / PAT / G** | caching hints, global flag — implementation-specific |

> [!tip] No "valid" bit on x86?
> x86 has only a present bit. P=0 means *either* invalid (no mapping) *or* swapped out. The OS distinguishes via its own auxiliary metadata. Many other ISAs have separate **valid** and **present** bits.

## How the MMU Uses Each Bit

```c
PTE = PageTable[VPN];

if (!PTE.Valid)             raise(SEGFAULT);    // unmapped
if (!PTE.Present)           pageFault();        // swapped, needs OS
if (op == WRITE  && !PTE.W) raise(PROT_FAULT);  // write to RO
if (op == EXEC   && !PTE.X) raise(PROT_FAULT);  // exec from non-X (NX)
if (mode == USER && !PTE.U) raise(PROT_FAULT);  // user touching kernel page

PTE.A = 1;                                       // set accessed
if (op == WRITE) PTE.D = 1;                      // set dirty

PA = (PTE.PFN << PAGE_SHIFT) | offset;
```

The hardware sets `A` and `D` automatically; the OS reads them to drive eviction and write-back decisions.

## Why the Granular Bits Exist

| Bit | Used by |
|---|---|
| Valid / Present | OS distinguishes mapped/unmapped/swapped |
| R/W/X | [[Isolation (OS)\|protection]]: NX bit defends against shellcode on stack/heap |
| U/S | kernel pages mapped into user address spaces (faster syscalls) without exposing them |
| A (accessed) | Clock / WSClock / LRU-approximation in [[Page Replacement\|replacement]] |
| D (dirty) | flush dirty pages back to disk on eviction; clean pages can just be discarded |

## PTE Size

Typically **4 bytes** (32-bit ISAs) or **8 bytes** (64-bit ISAs). 8-byte PTEs accommodate the wider PFN field on 64-bit systems plus extra flags (NX, etc.).

## Related Notes

- [[Page Table]] — the array of PTEs.
- [[Page Frame]] — what the PFN field selects.
- [[Page]] — the virtual-side counterpart.
- [[Paging]] — the umbrella scheme.
- [[Page Fault]] — triggered when present bit is 0.
- [[Page Replacement]] — uses A and D bits.
- [[TLB]] — caches PTEs.
- [[Ch 18 — Paging - Introduction]].
