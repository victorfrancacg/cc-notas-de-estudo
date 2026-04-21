---
title: Limited Direct Execution
tags:
  - concept
  - ostep
  - mechanism
  - virtualization
aliases:
  - LDE
type: concept
introduced_in: Ch 6
---

# Limited Direct Execution

> [!abstract] Definition
> **Limited Direct Execution (LDE)** is the technique for CPU virtualization where user code runs **natively** on the hardware (for speed) but with **hardware-enforced limits** (for safety and control). It is the single biggest mechanism underlying every modern OS's ability to time-share a CPU.

## The Two Words Pulled Apart

- **Direct** — the program runs on the real CPU at native speed. No interpretation, no emulation.
- **Limited** — user-mode hardware restrictions + traps + timer interrupts keep the OS in charge.

## What LDE Solves

| Challenge | LDE's answer |
|---|---|
| How to run user code fast? | Run it directly on the CPU. |
| How to stop it from doing forbidden things? | [[User Mode vs Kernel Mode|User mode]] — the hardware refuses privileged instructions. |
| How to let it request OS services? | [[System Call]] — trap into the kernel through pre-installed handlers. |
| How to reclaim the CPU from a runaway? | [[Timer Interrupt]] — hardware fires a timer regardless of what the program is doing. |
| How to switch processes? | [[Context Switch]] — save/restore register state around a process swap. |

## The Protocol Timeline

```
Boot (kernel mode):
  OS: initialize trap table     ─→  HW: remember syscall handler
  OS: start interrupt timer      ─→  HW: start timer (will fire in X ms)

Run (kernel mode):
  OS: create process, allocate mem, load program
      set up user stack, push argc/argv
      fill kernel stack with regs + PC
      return-from-trap
                                 ─→  HW: restore regs (from kstack)
                                       move to user mode
                                       jump to main()

Run (user mode):
  Program: Run main() ...
           Call a syscall → trap into OS
                                 ─→  HW: save regs (to kstack)
                                       move to kernel mode
                                       jump to trap handler

OS (kernel): handle the syscall, return-from-trap
                                 ─→  HW: restore regs (from kstack)
                                       move to user mode
                                       jump to PC after trap

Program (user): continues ...
                                 ─→  HW: timer interrupt fires
                                       save regs → kstack, kernel mode

OS (kernel): call switch() — save A's regs to PCB_A
                             restore B's regs from PCB_B
                             switch to B's kstack
             return-from-trap (into B)
                                 ─→  HW: restore B's regs, user mode
                                       jump to B's PC
```

## Two Layers of Register Saving

| Layer | Saved by | Where | When |
|---|---|---|---|
| **User registers** | Hardware (implicitly) | Kernel stack of the interrupted process | On any trap (syscall, timer, exception) |
| **Kernel registers** | Software (the `switch` routine) | [[Process Control Block]] of the process | Only when actually switching processes |

Getting this layering wrong is a classic kernel bug.

## What Makes This "Limited"

Without the limits, an OS would be "just a library" — no control, no isolation, no safety. The limits come from a handful of hardware features:

1. Two privilege modes.
2. A privileged instruction to install the trap table.
3. A privileged instruction to start the timer.
4. A privileged instruction to return-from-trap.
5. A trap instruction (unprivileged, callable from user code).

## The Analogy

> [!tip] Baby-proofing the machine
> The OS "baby-proofs" the hardware at boot: trap table installed, timer started. Then lets processes roam freely in user mode — knowing all the dangerous controls are behind protected doors that only the OS can open.

## Related Notes

- [[User Mode vs Kernel Mode]]
- [[Trap]], [[Trap Table]], [[System Call]]
- [[Timer Interrupt]], [[Interrupt]]
- [[Context Switch]]
- [[Cooperative vs Preemptive Scheduling]]
- [[Ch 6 — Mechanism - Limited Direct Execution]]
