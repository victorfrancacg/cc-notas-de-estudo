---
title: Dynamic Relocation
tags:
  - concept
  - ostep
  - memory
  - virtualization
aliases:
  - Hardware-Based Relocation
type: concept
introduced_in: Ch 15
---

# Dynamic Relocation

> [!abstract] Definition
> **Dynamic relocation** is the hardware-supported technique of translating a [[Process|process's]] [[Virtual Address|virtual addresses]] to physical addresses **at every memory reference, at runtime**, using per-process translation registers. The [[Base and Bounds]] mechanism is its simplest realization. Contrasts with **static relocation**, which patches addresses once at load time.

## Static vs Dynamic

| Static Relocation | Dynamic Relocation |
|---|---|
| Loader patches the binary at load time | Hardware translates each access at run time |
| Process can't move once placed | OS can relocate while paused |
| **No protection** — process can compute any physical address | Bounds register enforces protection |
| Older technique (e.g., MS-DOS .EXE relocations) | Modern technique (every Unix-y system) |

## What "Dynamic" Means

The same virtual address `0x100` in process A might map to physical `33024` while A runs the first time, then to `49408` after the OS swaps A out and back in to a different slot. The program never notices because the **base register** changes; the program's instructions and pointers are never rewritten.

## Why It Matters

- Enables [[Time Sharing]] efficiently — many processes resident at once, the OS picks slots.
- Foundation for every later VM mechanism: [[Segmentation]], [[Paging]], [[TLB]] all build on the runtime-translation idea.
- Provides the protection that [[Static Relocation]] could not.

## Mechanism Recap

See [[Base and Bounds]] for the full algorithm. In short:

```
physical = base + virtual
if (virtual >= bounds) trap
```

## Related Notes

- [[Base and Bounds]] — the simplest dynamic-relocation hardware.
- [[Address Translation]] — generalization beyond base/bounds.
- [[MMU]] — performs the translation.
- [[Memory Virtualization]] — what dynamic relocation enables.
- [[Ch 15 — Mechanism - Address Translation]].
