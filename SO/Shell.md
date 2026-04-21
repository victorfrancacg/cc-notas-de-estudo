---
title: Shell
tags:
  - concept
  - ostep
  - unix
  - process
aliases:
  - UNIX Shell
  - Shell Program
type: concept
introduced_in: Ch 5
---

# Shell

> [!abstract] Definition
> A **shell** is a user-level program that provides a command-line interface to the OS. It reads command lines, parses them, and (usually) creates child processes to run them via [[fork()]], [[exec()]], and [[wait()]].

There is nothing magical about it — a shell is just a UNIX program, typically written in C, using the process API like any other.

## Examples

`bash`, `zsh`, `tcsh`, `dash`, `fish`, `sh`. You can swap your shell freely — the kernel doesn't care which one you use.

## The Shell Command Loop

```
while (not EOF):
    print prompt
    read a line
    parse into (program, args, redirections, pipes)
    fork()
    if child:
        set up redirections / pipes
        exec(program, args)
    if parent:
        wait(child)
```

This handful of lines is essentially *the shell*. Everything else — history, autocompletion, scripting — is features built on top.

## Why the fork/exec Split Matters to the Shell

Between `fork()` and `exec()` the child is *still a copy of the shell*. This is the window where the shell can customize the child's environment:

- **[[I/O Redirection]]** — `cmd > file.txt`, `cmd < input`, `cmd 2>&1`.
- **[[Pipe (UNIX)|Pipes]]** — `cmd1 | cmd2`. Shell creates a pipe before forking, dup2s fds after forking, then execs.
- **Change working directory** (rarely — usually done via `cd` *before* fork).
- **Drop privileges**, set env vars, adjust resource limits (`ulimit`, `nice`).

Without this window, shells as we know them wouldn't exist. See also [[Mechanism vs Policy]]: the kernel gives a mechanism (fork, exec); the shell's policy is how to use them.

## Built-in Commands vs External Commands

Some commands are handled *inside* the shell without forking:

- `cd` — must run in the shell itself, because changing directory in a child wouldn't affect the parent shell.
- `exit`, `export`, `alias`, `source`, loops and conditionals in scripting.

External commands (`ls`, `gcc`, `grep`, …) are separate binaries invoked via `fork()`+`exec()`.

## A Concrete Redirection Trace

`wc p3.c > output.txt`:

1. Shell forks.
2. Child closes FD 1 (stdout).
3. Child opens `output.txt` — gets FD 1 (the lowest free slot).
4. Child execs `wc p3.c`.
5. `wc` writes to "stdout" (FD 1) — but that now points at `output.txt`.
6. Parent waits for child.

Magic without magic.

## Related Notes

- [[fork()]], [[exec()]], [[wait()]].
- [[I/O Redirection]], [[Pipe (UNIX)]].
- [[Ch 5 — Interlude - Process API]].
