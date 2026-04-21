---
title: User Mode vs Kernel Mode
tags:
  - concept
  - ostep
  - mechanism
  - architecture
aliases:
  - User Mode
  - Kernel Mode
  - Privilege Level
  - Ring 0
type: concept
introduced_in: Ch 6
---

# User Mode vs Kernel Mode

> [!abstract] Definition
> Hardware provides (at least) two CPU **execution modes**: a **restricted user mode** for application code, and a **privileged kernel mode** for the OS. The current mode determines which instructions the CPU will execute; privileged instructions in user mode trigger an exception.

## What Each Mode Can Do

| | User Mode | Kernel Mode |
|---|---|---|
| Regular arithmetic, loads, stores | Allowed | Allowed |
| Access user-space memory | Allowed | Allowed |
| Access kernel memory | **Denied** | Allowed |
| Issue I/O instructions directly | **Denied** | Allowed |
| Modify [[Trap Table]] | **Denied** | Allowed |
| Start/stop [[Timer Interrupt]] | **Denied** | Allowed |
| Change MMU / page tables | **Denied** | Allowed |
| Halt the CPU | **Denied** | Allowed |

Attempting a privileged instruction in user mode → hardware exception → trap into the kernel → OS typically kills the process.

## How Mode Changes

User → Kernel happens only via:

- **[[System Call]]** (voluntary) — user code issues a trap instruction.
- **Exception** (unexpected) — divide-by-zero, invalid memory access, illegal instruction.
- **Interrupt** (asynchronous) — from hardware (timer, I/O device).

Kernel → User happens only via:

- **Return-from-trap** instruction (itself privileged).

Both transitions are **atomic** — you can't end up stuck "half transitioned".

## Why Two Modes Aren't Enough for Everyone

Some architectures offer more. x86 has **four rings** (0, 1, 2, 3), though in practice Linux/Windows only use 0 (kernel) and 3 (user). Virtualization added ring -1 (hypervisor).

## Visual

```
+------------------------------------+
|          Kernel Mode (OS)          |
|  can do anything                   |
|                                    |
|  * I/O instructions                |
|  * Install trap table              |
|  * Start/stop timer                |
|  * Access any memory               |
+----^-------------------------^-----+
     |                         |
  trap /                      return-from-trap /
  interrupt /                 (kernel → user)
  exception
     |                         |
+----|-------------------------v-----+
|           User Mode (apps)         |
|  restricted subset of CPU          |
+------------------------------------+
```

## Why This Design Matters

> [!tip] Hardware enforces the wall
> The OS alone could never isolate processes — nothing prevents a process from overwriting OS memory or taking over the CPU. User mode *must* be enforced by the CPU itself. Without hardware support, there is no real OS.

## Related Notes

- [[Limited Direct Execution]] — relies crucially on the two modes.
- [[Trap]], [[System Call]], [[Trap Table]].
- [[Kernel]].
- [[Isolation (OS)]], [[Principle of Least Privilege]].
