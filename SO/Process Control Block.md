---
title: Process Control Block
tags:
  - concept
  - ostep
  - process
  - data-structure
aliases:
  - PCB
  - Process Descriptor
  - Task Struct
type: concept
introduced_in: Ch 4
---

# Process Control Block

> [!abstract] Definition
> A **Process Control Block (PCB)** — also called a **process descriptor** or **task struct** — is the kernel-side data structure that holds everything the OS needs to manage a single [[Process]].

## Why It Exists

When a process isn't running, *all* of its [[Machine State]] that's not already in its [[Address Space]] has to live somewhere. The PCB is that somewhere.

Without a PCB, the OS couldn't pause a process and resume it later — the details would be lost.

## What's Inside (typical fields)

| Field | What |
|---|---|
| **PID** | Process identifier. |
| **State** | Running, Ready, Blocked, Zombie, etc. See [[Process States]]. |
| **Register context** | Saved general-purpose registers, [[Program Counter|PC]], stack pointer. Only relevant when the process is *not* running. |
| **Memory info** | Pointers to the [[Address Space]] structures — page tables, size, mappings. |
| **Kernel stack** | Per-process kernel stack for handling syscalls / interrupts on behalf of this process. |
| **Parent pointer** | Who created me (needed for [[wait()]] semantics). |
| **Open files** | Array of file descriptors → file objects. |
| **Working directory** | Current directory (for relative paths). |
| **Trap frame** | CPU state captured on the most recent trap into the kernel. |

## The xv6 Example

From xv6 (a small teaching OS), simplified:

```c
// Saved registers for a non-running process
struct context {
    int eip, esp;
    int ebx, ecx, edx, esi, edi, ebp;
};

enum proc_state { UNUSED, EMBRYO, SLEEPING,
                  RUNNABLE, RUNNING, ZOMBIE };

struct proc {
    char            *mem;              // base of process memory
    uint             sz;               // size
    char            *kstack;           // kernel stack
    enum proc_state  state;
    int              pid;
    struct proc     *parent;
    void            *chan;             // sleep channel
    int              killed;
    struct file     *ofile[NOFILE];    // open files
    struct inode    *cwd;              // current dir
    struct context  *context;          // switch here to run
    struct trapframe *tf;
};
```

Real systems (Linux `task_struct`) are vastly larger — hundreds of fields tracking scheduling, namespaces, signals, cgroups, security contexts, etc. — but the core idea is the same.

## Where PCBs Live

In the [[Process List]] — a kernel data structure that tracks all PCBs. Typical implementations: linked list, hash by PID, or array in small OSes.

## What's *Not* Stored in the PCB

- **Code and data** — those live in the process's [[Address Space]], pointed to from the PCB.
- **Run-time stack / heap contents** — also in the address space.

The PCB is a *handle* to the process, not the whole process.

## Related Notes

- [[Process]], [[Process States]], [[Process List]].
- [[Context Switch]] — the act of swapping one PCB's register context out and another's in.
- [[Machine State]] — the abstract concept the PCB makes concrete.
