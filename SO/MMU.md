---
title: MMU
tags:
  - concept
  - ostep
  - memory
  - hardware
aliases:
  - Memory Management Unit
type: concept
introduced_in: Ch 15
---

# MMU (Memory Management Unit)

> [!abstract] Definition
> The **MMU** is the CPU subsystem that performs **[[Address Translation|address translation]]** on every memory reference: it takes a [[Virtual Address|virtual address]] from the CPU's pipeline, transforms it into a physical address, and forwards the request to the memory subsystem. It also enforces **protection** — out-of-bounds or permission-violating accesses raise a hardware fault.

## What the MMU Does (Per Memory Access)

For every load, store, and instruction fetch:

1. Receive a virtual address from the CPU pipeline.
2. Look up the translation — via base/bounds registers ([[Base and Bounds|simple case]]) or by walking [[Page Table|page tables]] (modern case).
3. Check permissions — read/write/execute, user/kernel.
4. If valid: emit the physical address to the memory bus.
5. If invalid: raise an exception ([[Segmentation Fault|segfault]], page fault).

## Evolution Through OSTEP Memory Virtualization (Ch 15–20)

| Chapter | Mechanism the MMU Implements |
|---|---|
| [[Ch 15 — Mechanism - Address Translation\|Ch 15]] | [[Base and Bounds]] — two registers |
| [[Ch 16 — Segmentation\|Ch 16]] | [[Segmentation]] — base/bounds per segment |
| [[Ch 18 — Paging - Introduction\|Ch 18]] | [[Paging]] — walk a page table |
| [[Ch 19 — Paging - Faster Translations (TLBs)\|Ch 19]] | Add a [[TLB]] cache |
| [[Ch 20 — Paging - Smaller Tables\|Ch 20]] | Walk multi-level / inverted page tables |

The MMU's hardware grows more elaborate, but the role stays constant: translate-and-check on every access.

## Why Translation Must Be in Hardware

Every load and store goes through it. Even nanoseconds of overhead per access would slow programs by orders of magnitude. The [[TLB]] exists precisely because **page-table walks are too slow to do every time** — caching the translation in the MMU itself is necessary.

## Software-Managed vs Hardware-Managed

- **Hardware-managed TLB** (x86, ARM): the MMU walks the page table itself on a TLB miss.
- **Software-managed TLB** (MIPS, some RISC): the MMU traps to the OS on a TLB miss; the OS code finds the translation and refills the TLB.

Both are real today. See [[TLB]] / [[Ch 19 — Paging - Faster Translations (TLBs)|Ch 19]].

## Related Notes

- [[Address Translation]] — the umbrella concept.
- [[Base and Bounds]], [[Segmentation]], [[Paging]] — successive translation schemes the MMU implements.
- [[TLB]] — the MMU's translation cache.
- [[Page Table]] — the data structure the MMU walks.
- [[Virtual Address]], [[Physical Address]] — its inputs and outputs.
- [[Memory Virtualization]] — what the MMU enables.
- [[Ch 15 — Mechanism - Address Translation]].
