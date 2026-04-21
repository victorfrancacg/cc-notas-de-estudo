---
title: Pipe (UNIX)
tags:
  - concept
  - ostep
  - unix
aliases:
  - UNIX Pipe
  - Anonymous Pipe
type: concept
introduced_in: Ch 5
---

# Pipe (UNIX)

> [!abstract] Definition
> A **pipe** is an in-kernel, unidirectional byte queue used to connect the output of one process to the input of another. Created by the `pipe()` system call, consumed via the two file descriptors it returns.

## The Canonical Example

```
prompt> grep -o "foo" file | wc -l
```

- `grep` writes "foo" to stdout.
- `wc` reads lines from stdin.
- The shell wires them: `grep`'s stdout → pipe → `wc`'s stdin.

## Signature

```c
#include <unistd.h>
int pipe(int fds[2]);
```

- `fds[0]` — **read** end.
- `fds[1]` — **write** end.

## How a Shell Wires `cmd1 | cmd2`

1. Shell calls `pipe(fds)`.
2. Shell `fork()`s for `cmd1`:
   - Child: `dup2(fds[1], STDOUT_FILENO)`; close both `fds`; `exec(cmd1)`.
3. Shell `fork()`s for `cmd2`:
   - Child: `dup2(fds[0], STDIN_FILENO)`; close both `fds`; `exec(cmd2)`.
4. Parent closes both ends of the pipe.
5. Parent `wait()`s for both children.

Closing unused ends is crucial — otherwise the reader never sees EOF and the whole pipeline hangs.

## Characteristics

- **Unidirectional.** Need two-way? Use two pipes, or a [[Socketpair]].
- **Blocking by default.** Writer blocks if pipe full; reader blocks if pipe empty.
- **Buffered in the kernel.** Linux default: 64 KB. Writes over this size split.
- **No persistence.** The pipe dies when all its FDs are closed. See [[Named Pipe (FIFO)]] for the file-system-resident variant.

## Why Pipes Matter

Pipes are the backbone of UNIX's **composability** philosophy: small programs doing one thing each, chained via text streams. Ken Thompson / Doug McIlroy's insight from 1973.

## Related Notes

- [[Shell]] — who wires pipes.
- [[I/O Redirection]] — sibling feature.
- [[fork()]], [[exec()]] — the mechanism window.
- [[File Descriptor]].
