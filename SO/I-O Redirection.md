---
title: I/O Redirection
tags:
  - concept
  - ostep
  - unix
aliases:
  - I/O Redirection
  - Output Redirection
  - Input Redirection
type: concept
introduced_in: Ch 5
---

# I/O Redirection

> [!abstract] Definition
> **I/O redirection** is a shell technique that reconnects a program's standard input, output, or error to a file — e.g., `cmd > out.txt`, `cmd < in.txt`, `cmd 2>&1`. It relies on the UNIX convention that `open()` returns the **lowest-numbered free file descriptor**.

## How It Works (Standard Output Case)

A UNIX process by default has:

| FD | Role |
|---|---|
| 0 | stdin |
| 1 | stdout |
| 2 | stderr |

To redirect stdout to a file, the shell:

1. `fork()`.
2. In the child: `close(1)` — FD 1 is now free.
3. In the child: `open("file.txt", O_WRONLY | O_CREAT | O_TRUNC, ...)` — since FD 1 is the lowest free slot, `open()` returns **1**.
4. In the child: `execvp(...)` — now when the program writes to FD 1, it writes to `file.txt`.
5. Parent waits.

The new program has no idea redirection happened — it just writes to FD 1 like always.

## Why It Works

Two UNIX design choices make redirection trivial:

1. **Uniform file-descriptor abstraction** — whether FD 1 is a terminal, a pipe, or a file, `write()` syntax is identical.
2. **FDs survive `exec()`** — the open-file table persists across the transition.

## Common Forms

| Syntax | Meaning |
|---|---|
| `cmd > file` | stdout to `file` (truncate). |
| `cmd >> file` | stdout appended to `file`. |
| `cmd < file` | stdin from `file`. |
| `cmd 2> file` | stderr to `file`. |
| `cmd > out 2>&1` | stdout to `out`, and stderr to same place as stdout. |
| `cmd1 | cmd2` | stdout of `cmd1` to stdin of `cmd2`. See [[Pipe (UNIX)]]. |

## Related Notes

- [[Shell]] — who implements redirection.
- [[fork()]], [[exec()]] — the mechanism window.
- [[Pipe (UNIX)]] — redirection's sibling.
- [[File Descriptor]].
