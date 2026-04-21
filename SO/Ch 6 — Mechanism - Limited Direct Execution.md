---
title: Ch 6 — Mechanism - Limited Direct Execution
tags:
  - ostep
  - chapter
  - part/virtualization
  - mechanism
type: chapter
book: OSTEP
chapter: 6
pages: 57-72
---

# Ch 6 — Mechanism: Limited Direct Execution

> [!abstract] One-sentence summary
> To virtualize the CPU efficiently and safely, the OS runs user programs **directly on the hardware** but with **guardrails**: two privilege levels ([[User Mode vs Kernel Mode|user/kernel]]), [[Trap|traps]] for controlled kernel entry, and a [[Timer Interrupt]] to periodically take back control.

## Crux of the Problem

> [!example] How to Efficiently Virtualize the CPU with Control?
> The OS must run user code fast (performance) *and* keep the ability to stop it, switch it out, and restrict what it can do (control). Two challenges that pull in opposite directions.

## 6.1 Basic Technique: [[Limited Direct Execution]] (LDE)

> [!tip] LDE in one line
> *Direct* execution = let the program run natively on the CPU (fast). *Limited* = hardware-enforced walls around what it can do and when it runs.

### Direct Execution (Without Limits)

| OS side | Program side |
|---|---|
| Create process entry | — |
| Allocate memory | — |
| Load program | — |
| Set up stack, registers | — |
| `call main()` | Run `main()` |
| — | `return` from main |
| Free memory, remove from process list | — |

Works for speed, fails for **everything else**: no control, no isolation, no way to switch processes. Hence the "limited" addition.

## 6.2 Problem #1: Restricted Operations

> [!example] Crux: How to Perform Restricted Operations
> A process must be able to do I/O and other restricted things **without** gaining complete control of the system.

### Hardware Solution — Two Modes

- **[[User Mode vs Kernel Mode|User mode]]** — most instructions allowed, but privileged ones (like direct I/O or installing a trap table) trigger an exception.
- **[[User Mode vs Kernel Mode|Kernel mode]]** — everything allowed. Only the OS runs here.

### Bridging the Gap — [[System Call|System Calls]] via [[Trap|Traps]]

To do something privileged, user code executes a **trap instruction**:

1. Hardware **saves** user registers (onto a per-process kernel stack).
2. Hardware **raises privilege** to kernel mode.
3. Hardware **jumps** to a predefined handler in the OS ([[Trap Table]]).
4. OS handles the request.
5. OS executes **return-from-trap**, which restores registers and lowers privilege.

System-call number identifies which call was made; OS validates it.

> [!warning] Trap tables are privileged — for good reason
> If user code could install its own trap table, it could hijack the kernel. Installing the trap table is a privileged instruction that fails in user mode. See [[Principle of Least Privilege]].

## 6.3 Problem #2: Switching Between Processes

> [!example] Crux: How to Regain Control of the CPU
> While a process runs, the OS is not running. How can the OS do anything — including switch away from a rogue process?

### Approach A — Cooperative

- Trust processes to make system calls frequently.
- Provide a `yield()` syscall for voluntary switching.
- **Fails** against infinite loops: only recourse is **reboot**. See [[Cooperative vs Preemptive Scheduling]].

### Approach B — Non-Cooperative (Timer Interrupt)

- Hardware fires a [[Timer Interrupt]] every few milliseconds.
- Interrupt halts the running process and jumps to the OS's interrupt handler.
- OS now has control; decides: keep running this process, or switch.
- Starting the timer is privileged — only the OS can (at boot).

This is **preemption**. Every modern OS relies on it.

## [[Context Switch]]

When the OS decides to switch, it:

1. Saves the current process's general-purpose registers + kernel SP + PC into its [[Process Control Block]] / kernel stack.
2. Loads the next process's saved state from its PCB.
3. Switches kernel stacks.
4. Returns from trap — hardware restores the next process's user registers.

Two register-save/restore layers happen:

- **User regs** — saved by *hardware* on trap entry, onto the kernel stack.
- **Kernel regs** — saved by *software* (OS's switch routine) into the PCB.

## 6.4 Worried About Concurrency?

> [!warning] Teaser for Part II
> If an interrupt fires in the middle of a syscall, the kernel is now handling two things at once. Naïve solutions: disable interrupts briefly, or use locking. Full treatment: [[Concurrency]] in Part II.

## 6.5 Summary — Baby-Proofing the CPU

> [!tip] The baby-proofing analogy
> The OS baby-proofs the machine at boot: installs trap handlers (hides the sharp corners) and starts the timer (sets a bedtime). Then it lets processes run freely in user mode, confident that the OS can always step in.

## How Long Does This Cost?

| Operation | ~1996 | Modern (GHz era) |
|---|---|---|
| System call (null) | ~4 µs | sub-µs |
| Context switch | ~6 µs | sub-µs |

Measured with `lmbench`. OS performance often tracks CPU speed — but not always memory bandwidth.

## Related Notes

- [[Limited Direct Execution]] — deep dive on the technique.
- [[User Mode vs Kernel Mode]], [[Trap]], [[Trap Table]], [[Timer Interrupt]], [[Context Switch]].
- [[Cooperative vs Preemptive Scheduling]].
- [[System Call]] — the call that triggers traps.
- [[Kernel]] — who runs in kernel mode.
- [[MOC - CPU Virtualization]].
