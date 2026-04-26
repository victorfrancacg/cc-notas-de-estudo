---
title: Orphan Process
tags:
  - concept
  - ostep
  - process
  - unix
type: concept
introduced_in: Ch 5
---

# Orphan Process

> [!abstract] Definition
> An **orphan process** is a [[Process]] whose parent has terminated *before* the child. The child is "orphaned" — its parent is gone, but the child keeps running. To prevent orphans from becoming unkillable, UNIX **re-parents** them to `init` (PID 1, or `systemd` on modern Linux), which routinely reaps any child that exits.

## How a Process Becomes an Orphan

```
parent → fork() → child
parent exits (without wait()ing)
                    │
                    ↓
              child still running
              child's parent now = PID 1 (init)
```

The kernel walks the parent's child list at exit and re-assigns each child to `init`. From then on, when the child exits, `init` will `wait()` on it — so the child does not linger as a [[Zombie Process|zombie]].

## Orphan vs Zombie

| Orphan | Zombie |
|---|---|
| Child still running, parent dead | Child exited, parent still alive but hasn't `wait()`ed |
| Re-parented to init | Stuck in process table until parent reaps |
| Eventually reaped by init | Becomes orphan if parent dies; init reaps |

The two states can compose: an orphan that exits before init notices is briefly an orphan-zombie, immediately reaped.

## Intentional Orphans — Daemons

The classic UNIX **daemon** idiom uses orphaning deliberately:

```c
pid_t pid = fork();
if (pid > 0) exit(0);   // parent exits — child is orphaned, re-parented to init
// child runs as a daemon, detached from any terminal session
setsid();               // new session, no controlling terminal
// ... daemon work ...
```

The detachment ensures the daemon survives terminal close, doesn't receive SIGHUP from a logout, and is owned by `init` (which never goes away).

This pattern is older than `systemd`-style service supervision but still appears.

## Modern Linux: `init` ≠ `init`

In modern systems:
- PID 1 is `systemd` (or, in containers, often a tiny stub like `tini` or `dumb-init`).
- The reaper logic is the same: any orphaned child gets re-parented to PID 1.
- In containers, if PID 1 doesn't reap, you get **zombie accumulation** — a common container-engineering pitfall fixed by using a proper init (tini, etc.).

## Related Notes

- [[Process]] — what gets orphaned.
- [[Zombie Process]] — the sibling state.
- [[fork()]], [[wait()]] — the API around the lifecycle.
- [[Process States]] — orphan isn't a formal state, but interacts with the lifecycle.
- [[Shell]] — uses orphan-and-detach for background jobs.
- [[Ch 5 — Interlude - Process API]].
