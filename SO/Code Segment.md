---
title: Code Segment
tags:
  - concept
  - ostep
  - memory
  - address-space
aliases:
  - Text Segment
  - .text
  - Program Code
type: concept
introduced_in: Ch 13
---

# Code Segment

> [!abstract] Definition
> The **code segment** (a.k.a. **text segment**, `.text` in ELF) is the region of a process's [[Address Space]] that holds the program's **machine instructions**. It is **static** in size, **read-only** at runtime, and conventionally lives at the **bottom** of the address space.

## Why It's Static and Read-Only

- The compiler/linker decided how many bytes of instructions the program has when the binary was built. The OS doesn't grow or shrink the code segment at run time.
- Marking it **read-only** is a [[Principle of Least Privilege|principle-of-least-privilege]] move: the program has no business overwriting its own instructions, so the hardware refuses to let it. An accidental wild write into `.text` traps cleanly instead of silently corrupting code.
- Read-only also enables **sharing**: every running instance of `bash` can share *one* physical copy of the bash code segment across all processes. The kernel maps the same physical pages into every bash process's address space.

## Position in the Address Space

```
   0KB  ┌──────────────────┐
        │ Program Code     │  static, read-only
   1KB  ├──────────────────┤
        │ Heap             │  grows ↓
        │     ↓            │
        │  (free)          │
        │     ↑            │
  15KB  │ Stack            │  grows ↑
  16KB  └──────────────────┘
```

Putting code at the *bottom* (low addresses) is convention, not requirement. ASLR (address-space layout randomization) often shifts it to a randomized base for security.

## What's Actually In the Code Segment

- Compiled function bodies (every `.o` file's `.text` linked together).
- Constant inline data the compiler chose to embed near instructions.
- Jump tables, dispatch tables (sometimes split into `.rodata`).
- The program's entry point (e.g., `_start`, which eventually calls `main`).

Other "static" regions that often sit near code but are *not* code:

- **`.rodata`** — read-only data (string literals, `const` globals).
- **`.data`** — initialized read-write globals.
- **`.bss`** — zero-initialized globals (no bytes in the binary, allocated at load time).

## Code Segment and Memory Virtualization

Code is mapped from the executable file via the kernel's loader (typically using a **file-backed [[mmap()]]**). Two consequences:

- **Demand-paged**: a code page is brought into RAM only when first executed. Cold startup is faster.
- **Discardable**: the kernel can evict a code page back to "the executable on disk" without writing anything (it's read-only). On reuse, it just re-reads from the binary.

> [!tip] Code pages are a free win for the page cache
> Because the OS knows where to refetch them, it can reclaim them under memory pressure with zero I/O cost on eviction.

## Related Notes

- [[Address Space]] — the code segment is one of three canonical regions.
- [[Heap (Runtime)]], [[Stack (Runtime)]] — the dynamic regions.
- [[Process]] — has exactly one code segment.
- [[exec()]] — replaces the code segment with that of a new program.
- [[mmap()]] — how the loader maps the executable's `.text` into memory.
- [[Memory Virtualization]] — what makes code-page sharing across processes possible.
- [[Principle of Least Privilege]] — code is read-only as a defense.
