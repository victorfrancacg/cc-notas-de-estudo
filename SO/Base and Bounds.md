---
title: Base and Bounds
tags:
  - concept
  - ostep
  - memory
  - virtualization
  - mechanism
aliases:
  - Base & Bounds
  - Base and Limit
  - Base/Bounds
type: concept
introduced_in: Ch 15
---

# Base and Bounds

> [!abstract] Definition
> **Base and bounds** is the simplest hardware mechanism for [[Address Translation|address translation]]: two CPU registers per process — `base` (where the address space starts in physical memory) and `bounds` (its size). Every memory reference adds `base` to the [[Virtual Address|virtual address]] and traps if it exceeds `bounds`. Also called **dynamic relocation**.

## The Algorithm

```text
physical_address = base + virtual_address
if (virtual_address >= bounds)
    raise(OUT_OF_BOUNDS)
```

That's the entire translation. Two adds (well, one add, one compare) per memory access — fast enough to be unmeasurable on real hardware.

## Worked Example

Process A's [[Address Space]] is 16 KB. The OS places it at physical 32 KB:

- `base = 32K`, `bounds = 16K`.
- A executes `load 0x100` (= 256). Hardware computes `32K + 256 = 33024`. Allowed (256 < 16K). Memory bus sees address `33024`.
- A executes `load 0x5000` (= 20480). Hardware compares `20480 >= 16K (16384)` → **trap**. Process killed.

## What Base and Bounds Buys You

- **[[Memory Virtualization|Virtualization]]** — every process can be linked at virtual address 0; the OS places it anywhere physical.
- **Relocation** — when the process is paused, the OS can *move* it to a different physical region by copying its memory and updating `base`.
- **[[Isolation (OS)|Protection]]** — `bounds` enforcement means a process literally cannot generate a physical address outside its own slot.

## What It Doesn't Buy You

> [!warning] [[Internal Fragmentation]]
> The slot is fixed-size and *contiguous*. Code/heap/stack each have to fit inside it, but the unused space between heap and stack still occupies physical memory — wasted bytes inside the slot. [[Segmentation|Ch 16]] generalizes base/bounds to fix this.

> [!warning] No sharing, no sparse address spaces
> One slot per process. Two processes running the same code can't share that code's pages. A 32-bit address space (4 GB) demands 4 GB of contiguous physical RAM, even if the process uses 4 MB. Hopeless beyond toy examples.

## Hardware Cost

Tiny. Two registers, an adder, a comparator. The MMU of the late-1950s [[Time Sharing]] machines already supported this. Today's CPUs include vastly more sophisticated translation hardware (page-table walkers, [[TLB]]s, multi-level tables) but the same interposition pattern.

## OS Responsibilities

- **Process creation** — find a free slot in physical memory; store base/bounds in the [[Process Control Block]].
- **[[Context Switch]]** — save outgoing base/bounds; load incoming.
- **Out-of-bounds trap handler** — typically: kill the process, free its slot, log.

## Why "Dynamic" Relocation?

Because the process is *moved* (relocated) at run time without the program knowing: the same virtual address `0x100` may map to physical `33024` now and `49408` later, and the program is none the wiser.

This contrasts with **static relocation**, an older technique where the loader patched the binary's instructions at load time so they referenced the right physical addresses. Static relocation provided no protection (the program could compute any address it wanted) and no relocation after start.

## Related Notes

- [[Dynamic Relocation]] — alias for this technique.
- [[MMU]] — the hardware that does the add+check.
- [[Address Translation]] — the umbrella concept.
- [[Internal Fragmentation]] — the unfixable downside.
- [[Segmentation]] — generalizes base/bounds per region of address space.
- [[Address Space]] — what base/bounds maps.
- [[Context Switch]] — when registers are swapped.
- [[Ch 15 — Mechanism - Address Translation]].
