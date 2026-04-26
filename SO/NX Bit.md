---
title: NX Bit
tags:
  - concept
  - ostep
  - memory
  - security
  - paging
aliases:
  - No-Execute Bit
  - XD Bit
  - W^X
type: concept
introduced_in: Ch 23
---

# NX Bit (No-Execute)

> [!abstract] Definition
> The **NX bit** is a [[Page Table Entry|PTE]] flag that marks a [[Page|page]] as **non-executable**: the [[MMU]] traps if the CPU tries to fetch instructions from it. Combined with making writable pages non-executable (the **W^X** policy: write XOR execute), it defeats classic shellcode-injection attacks where the attacker writes shellcode onto the stack or heap and jumps to it.

## What Hardware Provides

- **AMD**: introduced the bit as **NX** (no-execute) in 2003.
- **Intel**: same feature, named **XD** (execute-disable). Adopted shortly after.
- **ARM, RISC-V**: equivalents in their own PTE formats.

The MMU checks the NX bit on every instruction fetch. If set, the fetch raises a page fault (typically delivered as `SIGSEGV`).

## W^X (Write XOR Execute)

The defensive doctrine: **a page is either writable or executable, never both**.

| Page type | Writable | Executable |
|---|---|---|
| Code (`.text`) | no | **yes** |
| Read-only data (`.rodata`) | no | no |
| Initialized data (`.data`) | yes | no |
| Heap | yes | no |
| Stack | yes | no |

Under this policy, even if an attacker writes shellcode somewhere, they can't run it directly. The classic stack-smashing exploit dies.

## The Attack This Stops

Pre-NX, the canonical buffer overflow:

1. Attacker overflows a stack buffer.
2. Overwrites the saved return address with a pointer to the buffer itself.
3. Buffer contents are attacker-supplied **shellcode**.
4. Function returns → CPU executes shellcode on the stack.

With NX (and stack marked non-executable):

4'. CPU fetches from stack → NX trap → process killed before shellcode runs.

This was a huge win — for a few years.

## What NX Doesn't Stop

> [!warning] [[Buffer Overflow|ROP and return-to-libc]]
> Attackers responded with **return-oriented programming**: chain existing executable code (in `.text`, libc, etc.) via crafted return addresses. No injected code, no NX violation. ROP is the modern stack-smashing attack.
>
> The defense layered on top: [[ASLR]] randomizes where those gadgets are, making the attack harder.

> [!warning] JIT compilers complicate W^X
> JavaScript engines, JVMs, .NET — all generate code at runtime, requiring writable-then-executable pages. They use careful permission flips: write code, change to RX, execute. Bugs in this transition have been exploited (e.g., V8 type confusion → arbitrary write to JIT pages).

## Implementation

In the [[Page Table Entry]], a single bit (the NX bit) toggles execute permission. The kernel sets this bit on:
- All anonymous mappings (heap, stack) by default.
- All `mmap(PROT_READ|PROT_WRITE)` regions without `PROT_EXEC`.
- Read-only data segments.

The C compiler ensures stacks are non-executable via the `--noexecstack` linker flag (the default for years).

## Status Today

Universally enabled. Disabling NX ("`execstack`") is rare and typically a bug or legacy compatibility fix. Modern Linux: every process runs with NX active by default.

## Related Notes

- [[Buffer Overflow]] — the attack class NX targets.
- [[ASLR]] — complementary defense; needed because of ROP.
- [[Page Table Entry]] — where the NX bit lives.
- [[Code Segment]] — the page type that *needs* execute permission.
- [[Stack (Runtime)]] — NX makes stack pages non-executable by default.
- [[Linux Virtual Memory]] — implementation.
- [[Ch 23 — Complete Virtual Memory Systems]].
