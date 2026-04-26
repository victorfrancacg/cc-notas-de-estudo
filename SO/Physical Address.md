---
title: Physical Address
tags:
  - concept
  - ostep
  - memory
aliases:
  - PA
type: concept
introduced_in: Ch 13
---

# Physical Address

> [!abstract] Definition
> A **physical address** is an address on the actual memory bus, identifying a real byte in DRAM (or memory-mapped I/O). It is what the [[MMU]] produces after translating a [[Virtual Address|virtual address]]. Programs almost never see physical addresses directly — that's the [[Memory Virtualization|illusion]] memory virtualization is built on.

## How a PA Is Constructed Under Paging

```
+------+--------+        +------+--------+
| VPN  | offset |   →    | PFN  | offset |
+------+--------+        +------+--------+
   virtual                  physical
```

The [[Page Table]] supplies the PFN; the offset is unchanged. Concatenation gives the physical address. The bottom 12 bits (for 4 KB pages) are identical between VA and PA.

## Where PAs Appear

- **Memory bus** — DRAM controllers see only PAs.
- **Page table entries** — PFN field encodes the physical frame.
- **DMA descriptors** — devices doing direct memory access need PAs (or IOMMU translations).
- **Kernel boot code** — runs with translation off briefly; uses PAs.

## Where PAs Don't Appear

- **User programs** — print only virtual addresses (`%p`, `&x`).
- **Pointers in C/C++** — virtual.
- **Function call addresses** — virtual.

## Address Width

Physical address width can differ from virtual:

| ISA | Virtual width | Physical width |
|---|---|---|
| x86_64 | 48 bits (or 57 with LA57) | up to 52 bits |
| ARM64 | up to 48 bits | up to 48 bits |
| RISC-V Sv48 | 48 bits | up to 56 bits |

Physical can be **wider** than virtual to support more RAM than fits in a single address space (e.g., 64 GB physical with 32-bit virtual via [[PAE|PAE]] on legacy x86).

## Why Programs Don't See PAs

Three of the OSTEP goals depend on this opaqueness:

- **Transparency** — programs don't know they're being relocated.
- **Isolation** — process A can't even *name* process B's physical memory.
- **Flexibility** — the OS can swap, share, dedupe, migrate without breaking the program.

If user code could read physical addresses, none of these would hold.

## Related Notes

- [[Virtual Address]] — its counterpart.
- [[MMU]] — what translates VA → PA.
- [[Page Frame]] — physical pages identified by PFN.
- [[Page Table]] — holds the mappings.
- [[Memory Virtualization]] — the umbrella.
- [[Ch 18 — Paging - Introduction]].
