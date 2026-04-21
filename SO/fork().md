---
title: fork()
tags:
  - concept
  - ostep
  - process
  - syscall
  - unix
aliases:
  - fork
  - fork system call
type: concept
introduced_in: Ch 5
---

# fork()

> [!abstract] Definition
> `fork()` is the UNIX system call that creates a new process by **cloning** the calling process. After `fork()`, there are **two** processes running the same code — the original (**parent**) and the new one (**child**).

## Signature

```c
#include <unistd.h>
pid_t fork(void);
```

## Return Value

| Returns                 | Meaning                                                                         |
| ----------------------- | ------------------------------------------------------------------------------- |
| **`> 0`** (child's PID) | You are the **parent**.                                                         |
| **`0`**                 | You are the **child**.                                                          |
| **`-1`**                | `fork()` failed (usually out of process slots or memory); no child was created. |

## The Canonical Pattern

```c
int rc = fork();
if (rc < 0) {
    perror("fork");
    exit(1);
} else if (rc == 0) {
    // CHILD: do child stuff
    printf("child: pid=%d\n", (int)getpid());
} else {
    // PARENT: do parent stuff; rc == child's PID
    printf("parent of %d: pid=%d\n", rc, (int)getpid());
}
```

## What Gets Copied?

The child receives a **copy** of:

- The parent's [[Address Space]] (code, data, heap, stack) — logically; often implemented via **copy-on-write** to avoid paying the cost until someone writes.
- Register values (including [[Program Counter|PC]]).
- Open file descriptors (both processes now point to the same underlying file entries).
- Current working directory, user ID, environment.

What does *not* carry over identically:

- PID — child gets a new one.
- Parent PID (PPID) — child's is the parent's PID.
- Pending signals, file locks, timers — see POSIX for details.

## Non-Determinism

> [!warning] Either process can run first
> After `fork()`, the OS [[Scheduler]] picks who runs next — could be parent, could be child. Output order is **not guaranteed**. Use [[wait()]] if you need to enforce ordering.

## What fork() Alone Is Good For

- Making multiple copies of *the same* program (web server accept loops, parallel computation).
- Pair `fork()` with [[exec()]] to run a *different* program in the child.

## Common Pitfalls

> [!warning] Classic fork() bugs
> - **Forgetting to check `rc == 0`** — doing parent-work in the child (or vice-versa).
> - **Forgetting to [[wait()]]** — leaving [[Zombie Process|zombies]] accumulating.
> - **Flushing stdio buffers twice** — if the parent had pending buffered output, *both* processes now have it and both flush it. Call `fflush(stdout)` before `fork()` if you care.
> - **Interaction with threads** — forking a multi-threaded program only clones the calling thread; locks held by other threads are now permanently held in the child. Prefer [[exec()]] quickly after fork in a threaded program.

## Related Notes

- [[exec()]], [[wait()]] — the rest of the triad.
- [[Process]], [[Process Control Block]].
- [[Shell]] — the #1 user of `fork()`.
- [[Zombie Process]], [[Orphan Process]].
- [[Ch 5 — Interlude - Process API]].
