---
title: Signal (UNIX)
tags:
  - concept
  - ostep
  - unix
  - process
aliases:
  - UNIX Signal
  - Signals
  - SIGINT
  - SIGKILL
type: concept
introduced_in: Ch 5
---

# Signal (UNIX)

> [!abstract] Definition
> A **signal** is a small asynchronous notification delivered by the kernel to a process. Signals can be sent by the kernel itself (on hardware faults, child exits, timers), by other processes via `kill()`, or by the user via keyboard shortcuts (Ctrl-C, Ctrl-Z).

## Canonical Signals

| Signal | Default action | Common trigger |
|---|---|---|
| `SIGINT` (2) | Terminate | Ctrl-C |
| `SIGQUIT` (3) | Core-dump + terminate | Ctrl-\\ |
| `SIGKILL` (9) | Terminate (**uncatchable**) | `kill -9 PID` |
| `SIGTERM` (15) | Terminate (catchable) | `kill PID` (default) |
| `SIGTSTP` (20) | Stop | Ctrl-Z |
| `SIGCONT` (18) | Continue stopped process | `fg`, `bg` |
| `SIGCHLD` (17) | Ignored | Child exited |
| `SIGSEGV` (11) | Terminate + core | Bad memory access |
| `SIGPIPE` (13) | Terminate | Write to a pipe with no readers |

## Syscalls

```c
int kill(pid_t pid, int sig);       // send a signal
sighandler_t signal(int sig, handler);  // legacy: register handler
int sigaction(int sig, ..., ...);   // modern: register handler with options
```

## Delivery Behavior

- **Asynchronous.** The process is interrupted wherever it is, the handler runs, then control returns (or the process dies).
- **Limited in handlers.** Most library functions are unsafe inside signal handlers. Use only **async-signal-safe** functions (e.g., `write()` yes, `printf()` no).
- **SIGKILL and SIGSTOP cannot be caught, blocked, or ignored** — they are the OS's emergency brakes.

## Process Groups

A signal can target:

- A single process (`kill(pid, ...)`).
- A **process group** (`kill(-pgid, ...)`). The shell uses this to deliver Ctrl-C to all processes in a pipeline at once.

## Example Usage

```c
#include <signal.h>
void handle_sigint(int sig) {
    const char msg[] = "caught SIGINT\n";
    write(STDERR_FILENO, msg, sizeof(msg) - 1);
}
signal(SIGINT, handle_sigint);
```

## Why Signals Exist

Signals are the OS's answer to: *how do you tell a running process something without waiting for it to check?* They're crude (no payload beyond the signal number, classic API) but ubiquitous.

> [!info] Richer alternatives
> Modern systems offer `signalfd()`, real-time signals with payloads, and event loops via `epoll`/`kqueue`. For most user code, classic signals suffice.

## Related Notes

- [[fork()]], [[Shell]].
- [[Process States]] — signals move processes between states (e.g., SIGSTOP → Blocked).
- [[Interrupt]] — hardware cousin, different layer.
