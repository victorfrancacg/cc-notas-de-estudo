---
title: Meltdown and Spectre
tags:
  - concept
  - ostep
  - memory
  - security
  - hardware
aliases:
  - Meltdown
  - Spectre
  - KPTI
type: concept
introduced_in: Ch 23
---

# Meltdown and Spectre

> [!abstract] Definition
> **Meltdown** and **Spectre** (disclosed January 2018) are families of CPU side-channel vulnerabilities exploiting **speculative execution**. The CPU guesses ahead, executing instructions whose effects it later discards if the guess was wrong — but **traces** of the speculation linger in the cache. Attackers measure cache timing to infer values that the CPU "shouldn't" have read, including kernel memory and other processes' data. The mitigation, **KPTI** (Kernel Page-Table Isolation), forced the OS to undo a long-standing optimization.

## The Background: Speculative Execution

Modern CPUs are massively pipelined. Rather than wait for branch decisions or memory accesses to complete, they **speculate**:

```
if (condition_that_takes_100_cycles)
    x = a[i];   // CPU starts loading a[i] before "condition" resolves
else
    x = b[i];   // CPU might also start this; commits whichever wins
```

If the speculation was right, the result is committed and the program runs faster. If wrong, the CPU **rolls back** the architectural state — registers, memory writes — as if the speculation never happened.

But: **caches are not rolled back**. If the speculation loaded a memory location, that location is now in cache, even after rollback. The attacker can detect this via timing.

## Meltdown

Reads kernel memory from user space.

```c
char *kernel_ptr = 0xFFFFFFFF80000000;
char value;
char probe[256 * 4096];

// Speculatively read kernel memory, then use it as an index:
value = *kernel_ptr;          // illegal — but speculation runs anyway
unused = probe[value * 4096]; // touches a probe-array slot dependent on value

// CPU eventually traps on the illegal kernel read and rolls back.
// But probe[value * 4096] is now cached.

// Attacker times each probe[i * 4096]:
for (int i = 0; i < 256; i++)
    measure_access_time(&probe[i * 4096]);
// The fast access reveals the kernel byte's value.
```

**Meltdown** specifically affected Intel CPUs: speculation didn't enforce kernel/user isolation strictly. The fix in software is **KPTI** (see below).

## Spectre

A larger family that doesn't require crossing privilege boundaries — abuses **branch prediction** to make the victim execute attacker-chosen speculative code paths.

```c
// In the victim:
if (i < array_size) {
    char x = array1[i];          // bounds-checked... right?
    char y = array2[x * 4096];   // probe array
}
```

If the attacker repeatedly trains the branch predictor to expect `i < array_size`, then provides an `i` *out* of bounds, the CPU **speculatively** executes the body — performing an out-of-bounds read of `array1[i]` and using its value to touch the probe array. Architectural rollback happens, but the cache doesn't forget.

Spectre is **architecture-independent** in concept and far harder to fix.

## Why The OS Mattered

Pre-Meltdown, every modern OS mapped **all kernel memory into every user process's upper half** — a deliberate optimization so that syscalls didn't need to switch page tables. The user couldn't *legitimately* read those pages (the U/S protection bit blocked it), so the mapping was "safe".

Meltdown shattered this: speculation could read kernel memory **even with the protection bit**, leak the value via the cache, and the rollback was too late.

## KPTI — Kernel Page-Table Isolation

The fix: **don't map kernel memory into user processes** at all. Each process has two page tables:

- **User table** — only user mappings + the bare minimum kernel stub for syscall entry/exit.
- **Kernel table** — full kernel mappings.

On every syscall and interrupt, the OS swaps to the kernel table and back. **Costs:**

- ~5–30% slowdown on syscall-heavy workloads.
- TLB flushes on each switch (mitigated with PCID/ASID).

> [!warning] KPTI was a step backwards in performance
> Decades of "kernel-in-every-user-process" optimization undone in a kernel patch. Meltdown was a watershed moment showing hardware-level threats can force OS-level redesigns.

## The Aftermath

- **Spectre v1, v2, v4** — many variants; mitigations include compiler flags (`retpoline`), microcode updates, branch-prediction barriers.
- **Hardware fixes** — newer Intel (post-Cascade Lake) and AMD CPUs have hardware mitigations.
- **Performance research** — entire papers on minimizing KPTI overhead; "speculation hiding" remains an active research area.

## Lessons

- **Hardware speculation is not transparent.** Architectural rollback ≠ cache rollback.
- **Side channels are real attack surface.** Cache timings, branch-prediction state, even DRAM rowhammer leak information.
- **Hardware-OS coupling is tighter than it looks.** Bug-for-bug compatibility goes both ways.

## Related Notes

- [[ASLR]] — was supposed to make kernel addresses unguessable; speculation defeats this.
- [[Linux Virtual Memory]] — implementation of KPTI.
- [[TLB]] — TLB flushes are part of KPTI's cost.
- [[Page Table]] — KPTI requires two of them per process.
- [[Buffer Overflow]] — older attack class; speculation is its modern cousin.
- [[Ch 23 — Complete Virtual Memory Systems]].
