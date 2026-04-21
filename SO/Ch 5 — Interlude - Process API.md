---
title: Ch 5 — Interlude - Process API
tags:
  - ostep
  - chapter
  - part/virtualization
  - process
  - api
type: chapter
book: OSTEP
chapter: 5
pages: 41-55
---

# Ch 5 — Interlude: Process API

> [!abstract] One-sentence summary
> UNIX creates processes with two syscalls — **[[fork()]]** and **[[exec()]]** — deliberately split so that a parent can customize the environment of the child *between* cloning and running the new program. [[wait()]] lets a parent collect a child's exit status. The shell is the canonical consumer.

## Crux of the Problem

> [!example] How to Create and Control Processes?
> What interfaces should the OS present for process creation and control? How should they be designed for **powerful functionality**, **ease of use**, and **high performance**?

## The Strange, Powerful Triad

| Call | What it does |
|---|---|
| [[fork()]] | Create a near-identical **copy** of the calling process. |
| [[exec()]] | Replace the current process image with a different program. |
| [[wait()]] | Pause until a child process finishes; collect its exit code. |

## 5.1 fork()

- Creates a child process.
- Child is (nearly) an exact copy: same memory contents (own copy though!), same register state — *almost*. Specifically, `fork()` returns **different** values to parent and child:
  - **Parent** gets the child's PID (> 0).
  - **Child** gets **0**.
  - **Both** get **-1** on failure.
- After `fork()`, you have **two** processes executing the instruction after `fork()`.
- Execution order is **nondeterministic**: either parent or child may run first. The [[Scheduler]] decides.

## 5.2 wait()

- A parent calls `wait()` to block until one of its children finishes.
- Returns the child's PID and exit code (via pointer argument).
- Closely related: [[waitpid()]] — wait for a *specific* child, or apply options.
- Using `wait()` after `fork()` makes output deterministic in parent/child ordering.

## 5.3 exec()

- Loads a new program (code + static data) into the current process's memory, **overwriting** the existing code. Heap and stack are re-initialized.
- It does **not** create a new process — it transforms the *current* one.
- A successful `exec()` **never returns** — the process is now a different program.
- Variants: `execl`, `execlp`, `execle`, `execv`, `execvp`, `execvpe`. They differ on how arguments / env / PATH lookup are handled.

## 5.4 Why? The Motivation (Lampson's Law — "Get it Right")

> [!tip] Why separate fork and exec?
> Because the gap between them is where **[[I/O Redirection]]** and **[[Pipe (UNIX)|pipes]]** live.
>
> After `fork()`, the child is *not yet* running the target program. The child can:
> 1. Close / reopen file descriptors (implementing `>`, `<`, `2>`).
> 2. Connect pipes between siblings (implementing `|`).
> 3. Change user ID, working directory, etc.
> 4. *Then* call `exec()`.
>
> A combined `spawn(program, args)` API couldn't offer this flexibility. Dividing the atom is what gives [[Shell|shells]] their power.

## The [[Shell]] in Action

```
prompt> wc p3.c > newfile.txt
```

What happens inside the shell:

1. Parse the command line.
2. `fork()`.
3. In the child: `close(STDOUT_FILENO)`, then `open("newfile.txt", ...)`. Since `open()` picks the lowest free file descriptor (now 1), STDOUT is redirected.
4. In the child: `execvp("wc", args)` — child becomes `wc`.
5. In the parent: `wait()` for the child.
6. Prompt again.

`wc`'s output now flows to `newfile.txt`.

## 5.5 Process Control & Users

- **[[Signal (UNIX)|Signals]]** — asynchronous notifications via `kill()`. Common: `SIGINT` (Ctrl-C), `SIGTSTP` (Ctrl-Z), `SIGCONT`, `SIGKILL`, `SIGTERM`.
- **`signal()`** — register a handler for a signal.
- **Process groups** — send a signal to many processes at once.
- **Users** — every process has an owner UID; users can only signal their own processes.
- **Superuser / root** — can do anything. Be sparing. "With great power comes great responsibility."

## 5.6 Useful Tools

| Tool | What |
|---|---|
| `ps` | List processes. |
| `top` | Live view of CPU / memory usage. |
| `kill` | Send signals. |
| `killall` | Kill by name. |

## Nuance — Should fork() Always Exist?

> [!warning] Not everyone agrees fork() is perfect
> A HotOS '19 paper (*"A fork() in the road"*) argues fork is a footgun at scale — poor interaction with threads, huge memory-doubling, inefficient in containers. Alternative: `spawn()`. OSTEP respects the critique but keeps teaching fork because it's what UNIX actually does.

## Related Notes

- [[fork()]], [[exec()]], [[wait()]] — the three syscalls, deep dives.
- [[Shell]] — the canonical consumer.
- [[I/O Redirection]], [[Pipe (UNIX)]] — tricks built between fork and exec.
- [[Signal (UNIX)]], [[Zombie Process]], [[Orphan Process]].
- [[MOC - CPU Virtualization]].
