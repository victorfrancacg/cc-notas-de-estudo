---
title: ASLR
tags:
  - concept
  - ostep
  - memory
  - security
aliases:
  - Address Space Layout Randomization
  - KASLR
type: concept
introduced_in: Ch 23
---

# ASLR (Address Space Layout Randomization)

> [!abstract] Definition
> **ASLR** is a security technique that randomizes the base addresses of code, libraries, heap, and stack within a process's virtual [[Address Space]] at process start. It makes exploits harder by preventing attackers from hard-coding target addresses — every run, the layout is different. **KASLR** applies the same idea to kernel mappings.

## What Gets Randomized

| Region | Randomized? |
|---|---|
| Main executable | yes (with PIE — position-independent executable) |
| Shared libraries | yes |
| Heap base | yes |
| Stack base | yes |
| `mmap()` allocations | yes |
| `vDSO` (kernel fast-syscall page) | yes |

Each shifts by a random offset within an architecture-specific range.

## Demonstration

```c
int main(int argc, char *argv[]) {
    int stack = 0;
    printf("%p\n", &stack);
    return 0;
}
```

```
$ ./random
0x7ffd3e55d2b4
$ ./random
0x7ffe1033b8f4
$ ./random
0x7ffe45522e94
```

Each run, the stack lives somewhere else. Without ASLR (older systems, or when disabled): **same address every run** — a gift to attackers.

## Why It Helps

Most exploits need to know **target addresses**:
- Where is `system()` in libc? (return-to-libc attack).
- Where are useful gadgets? ([[Buffer Overflow|ROP]]).
- Where is the saved return address on the stack?

With ASLR, an exploit using hard-coded addresses crashes (jump to garbage). The attacker now needs an **information leak** to discover the layout — which is a higher bar.

## What It Doesn't Stop

> [!warning] Information leaks bypass ASLR
> A bug that prints any pointer, or that lets the attacker read process memory (e.g., format-string vulnerabilities, out-of-bounds reads), reveals enough of the layout to defeat ASLR. ASLR is **a hurdle**, not a fix.

> [!warning] Page-aligned brute force on small entropy
> 32-bit ASLR has limited entropy (16 bits or so) — attackers can sometimes brute-force address guesses. 64-bit has much more room and is more resistant.

> [!warning] Speculative-execution attacks (Meltdown/Spectre)
> Attackers can infer kernel layout via speculation side channels. KASLR mitigations had to be supplemented by KPTI (Kernel Page-Table Isolation).

## KASLR — Kernel ASLR

The same idea applied to kernel mappings: the kernel's base address is randomized at boot. Defeats attacks that need to call kernel functions or read kernel data structures at known offsets.

KASLR's main weakness was speculative-execution side channels — fixed by **KPTI** (separate kernel page tables, not just randomized mappings).

## Implementation

The OS, when loading a binary or library:
1. Picks a random offset within an allowed range.
2. Maps the segments at `base_address + offset`.
3. Updates the dynamic linker / executable headers accordingly.

For PIE binaries (compiled with `-fPIE -pie`), this just works — the binary runs correctly at any base. Non-PIE binaries can only randomize libraries, heap, and stack, not the main executable's code.

## Status Today

ASLR is **on by default** in every major OS:
- Linux: `kernel.randomize_va_space = 2` (full).
- Windows: ASLR + Mandatory ASLR + High-entropy ASLR.
- macOS: ASLR + KASLR.

Disabling ASLR (`echo 0 > /proc/sys/kernel/randomize_va_space`) is a **debugging tool only** — never for production.

## Related Notes

- [[Buffer Overflow]] — the attack ASLR raises the bar against.
- [[Linux Virtual Memory]] — implementation.
- [[Address Space]] — what gets randomized.
- [[Meltdown and Spectre]] — speculative-execution flaws that bypass ASLR/KASLR.
- [[Ch 23 — Complete Virtual Memory Systems]].
