---
title: wait()
tags:
  - concept
  - ostep
  - process
  - syscall
  - unix
aliases:
  - wait
  - waitpid
type: concept
introduced_in: Ch 5
---

# wait()

> [!abstract] Definition
> `wait()` is the UNIX system call a **parent** uses to pause its execution until one of its **children** terminates, and to collect that child's exit status (reaping a [[Zombie Process|zombie]]).

## Signatures

```c
#include <sys/wait.h>

pid_t wait(int *status);
pid_t waitpid(pid_t pid, int *status, int options);
```

- `wait(&status)` — block until *any* child exits.
- `waitpid(pid, &status, opts)` — wait for a **specific** child, or with options:
  - `WNOHANG` — return immediately if no child has exited (poll, don't block).
  - `WUNTRACED` — return for stopped (Ctrl-Z'd) children too.

## Return Value

- The **PID** of the child that exited.
- `-1` on error (e.g., no children to wait for).

## The Status Argument

Passed by pointer; decoded with macros:

| Macro | Meaning |
|---|---|
| `WIFEXITED(status)` | Child exited normally. |
| `WEXITSTATUS(status)` | Returns the exit code (lower 8 bits). |
| `WIFSIGNALED(status)` | Child was killed by a signal. |
| `WTERMSIG(status)` | Which signal. |
| `WIFSTOPPED(status)` | Child was stopped (not exited). |

## Why It Exists

Two reasons:

1. **Synchronization** — the parent often needs to know when the child's work is done before proceeding.
2. **Cleanup** — without `wait()`, exited children become [[Zombie Process|zombies]] and their PCB slot leaks.

## A Shell's Use of wait()

```c
pid_t child = fork();
if (child == 0) {
    execvp(prog, args);
    exit(127);           // exec failed
}
// parent
int status;
waitpid(child, &status, 0);  // block until command finishes
// then print prompt again
```

This is exactly what your shell does between printing prompts.

## Caveats

> [!warning] Partial-wait semantics
> Sometimes `wait()` returns before the child exits — e.g., if the child is *stopped* (Ctrl-Z) rather than terminated, and you requested that via options. Read the man page. OSTEP's footnote: "beware of any absolute and unqualified statements this book makes".

> [!warning] If the parent dies first
> The child is re-parented to `init` (PID 1). `init` does `wait()` in a loop, so [[Orphan Process|orphans]] don't accumulate.

## Related Notes

- [[fork()]], [[exec()]] — the rest of the triad.
- [[Zombie Process]] — what `wait()` reaps.
- [[Orphan Process]] — what happens without a waiting parent.
- [[Ch 5 — Interlude - Process API]].
