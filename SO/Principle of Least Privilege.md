---
title: Principle of Least Privilege
tags:
  - concept
  - ostep
  - design-principle
  - security
aliases:
  - Least Privilege
  - POLP
type: concept
introduced_in: Ch 6
---

# Principle of Least Privilege

> [!abstract] Definition
> The **Principle of Least Privilege (PoLP)** states: **a component (process, user, module) should be given only the minimum authority needed to do its job**, and no more. Narrower privilege reduces the blast radius when something goes wrong.

Classic statement: Saltzer & Schroeder's 1975 paper *The Protection of Information in Computer Systems*.

## How the OS Applies It

The OS is shot through with least-privilege choices:

### User Mode

Ordinary processes run in [[User Mode vs Kernel Mode|user mode]] — most privileged instructions unavailable. They need kernel assistance (via [[System Call|syscalls]]) for anything touching the hardware.

### Restricted Operations

Installing the [[Trap Table]], starting the [[Timer Interrupt]], changing page tables, executing I/O directly — all are **privileged**. User code can't do them. Only the [[Kernel]] can.

### Per-User Permissions

On UNIX:

- Regular users control only their own processes.
- File permissions (owner/group/other, read/write/execute) restrict what files you can touch.
- The **superuser** ("root") can do anything — and should be used sparingly.

### Process Control

As OSTEP Ch 5 puts it:

> Who can send a signal to a process, and who cannot? Generally, the systems we use can have multiple people using them at the same time; if one of these people can arbitrarily send signals such as SIGINT (to interrupt a process, likely terminating it), the usability and security of the system will be compromised.

A user can kill their own processes but not someone else's. Least privilege enforced.

## Why It Matters Beyond Security

PoLP isn't just about malice — it's about **blast radius**:

- If a bug lets a process corrupt memory, it can only corrupt its own address space (assuming isolation).
- If a bug lets a process issue I/O it shouldn't, the system rejects it via traps.
- Failures are *contained*.

A violation of least privilege is how small bugs become catastrophic security vulnerabilities.

## Practical Implications

> [!tip] When you design something, ask:
> What's the minimum privilege this piece needs to work? Give it that and nothing more.

Examples:

- Run daemons as unprivileged users.
- Use `seteuid`/`setuid` to drop privileges after initialization.
- Containerize / sandbox untrusted code.
- Use capabilities (Linux fine-grained POSIX capabilities) instead of full root.
- Use read-only mounts where write isn't needed.

## Related Notes

- [[User Mode vs Kernel Mode]], [[System Call]], [[Trap]].
- [[Isolation (OS)]] — enabled by least privilege.
- [[OS Design Goals]].
- [[Operating System]].
