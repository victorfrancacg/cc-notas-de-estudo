---
title: exec()
tags:
  - concept
  - ostep
  - process
  - syscall
  - unix
aliases:
  - exec
  - execve
  - execvp
type: concept
introduced_in: Ch 5
---

# exec()

> [!abstract] Definition
> `exec()` is the family of UNIX system calls that **replaces** the current process's memory image with a different program. It does **not** create a new process — it transforms the caller into a different program.

## The Exec Family

All variants ultimately call the kernel-level `execve`. They differ in how arguments are passed:

| Variant | Args style | Searches `PATH`? | Passes env? |
|---|---|---|---|
| `execl` | **L**ist (`"arg0", "arg1", NULL`) | No | Parent's env |
| `execlp` | List | **P**ath-searched | Parent's env |
| `execle` | List | No | **E**xplicit env |
| `execv` | **V**ector (`argv[]`) | No | Parent's env |
| `execvp` | Vector | Path-searched | Parent's env |
| `execvpe` | Vector | Path-searched | Explicit env |

## What Happens on a Successful exec()

1. Kernel finds the executable (e.g., walks `PATH` for `execvp`).
2. Reads the new program's code + static data.
3. **Overwrites** the caller's [[Address Space]]:
   - Code segment → new program.
   - Heap and stack → re-initialized.
   - Uninitialized data (bss) → zeroed.
4. Sets [[Program Counter|PC]] to the new program's entry point.
5. Starts execution with the specified `argv` and environment.

> [!warning] exec() does not return on success
> If you see code after an `exec()` call, it's only running because `exec()` **failed**. The next thing after `exec()` should usually be a `perror` + `exit`.

## What Survives exec()?

| Preserved | Reset |
|---|---|
| **PID** (the process is the same!) | Code, static data |
| Parent relationship | Heap, stack |
| Open file descriptors (by default) | Registers |
| Working directory | Signal handlers (restored to defaults) |
| User/group IDs | Pending signals (mostly cleared) |

This preservation of **file descriptors across exec()** is what makes [[I/O Redirection]] work — the shell can set up redirections in the child (post-fork, pre-exec), and they stick after `exec()`.

## Canonical Pattern With fork()

```c
int rc = fork();
if (rc == 0) {
    // CHILD
    char *args[] = {"wc", "file.txt", NULL};
    execvp(args[0], args);
    // only reached on failure
    perror("exec");
    exit(1);
}
// PARENT continues; typically wait()s
```

## Why Not Just One Call That Forks-and-Execs?

A combined `spawn(prog, args)` would save a couple of lines, but it would close the door on everything a shell does **between** fork and exec: redirect I/O, set up pipes, change user/UID, change cwd, drop privileges.

Separating them is what [Lampson's law "Get it right"](https://pages.cs.wisc.edu/~remzi/Classes/537/Papers/lampson-hints.pdf) is about.

## Related Notes

- [[fork()]], [[wait()]] — the rest of the triad.
- [[Shell]] — runs a fork/exec pair for every command.
- [[I/O Redirection]], [[Pipe (UNIX)]].
- [[Ch 5 — Interlude - Process API]].
