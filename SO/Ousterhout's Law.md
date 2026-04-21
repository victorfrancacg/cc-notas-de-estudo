---
title: Ousterhout's Law
tags:
  - concept
  - ostep
  - design-principle
aliases:
  - Voodoo Constants
  - Avoid Voodoo Constants
type: concept
introduced_in: Ch 8
---

# Ousterhout's Law

> [!abstract] The principle
> **Avoid voodoo constants.** A "voodoo constant" is a magic parameter whose correct value is guessed rather than derived — it happens to work in testing, nobody knows why it's *the* right number, and changing it subtly breaks things. Avoid them when you can; document and expose them when you can't.

Named for **John Ousterhout** (creator of Tcl, author of *A Philosophy of Software Design*).

## Why They're a Problem

- **Unjustified.** Nobody can explain why *this* value and not another.
- **Workload-dependent.** What works for interactive desktops may starve a batch cluster.
- **Configuration-fossil.** Ends up in a config file with default values that *almost* nobody ever tunes.
- **Machine learning targets.** A growing trend is to auto-tune them via ML, which works but also admits we don't understand the system.

## The Canonical OS Example

The priority-boost period *S* in [[MLFQ]]:

- Too high (e.g., 10 s): long jobs starve for up to 10 s at a time.
- Too low (e.g., 10 ms): boosting is so frequent that interactive prioritization becomes meaningless.
- Just right: ??? workload-dependent.

So MLFQ exposes *S* as a tuning knob — and hopes sysadmins are wise, which they often aren't.

## Other Voodoo Constants You'll Meet in OSTEP

- Page replacement "aging" intervals.
- File-system write-back timers.
- Disk scheduler cost parameters.
- Lock-spinning iteration counts.

When you meet one, ask: "what evidence justifies this value?" If the answer is "it was the value in the previous version", you've found a voodoo constant.

## Tip — Use Advice Where Possible

When the OS can't derive the right value, let the user / admin / application provide **advice** (hints):

- UNIX `nice` — user hint for process priority.
- `madvise()` — app hint for memory access patterns.
- `posix_fadvise()` — file access hints.
- `ulimit` / `rlimit` — user-supplied resource ceilings.

The OS doesn't have to *obey* advice — but it can take it into account.

## The Broader Lesson

Ousterhout's Law is a small piece of a bigger principle: **systems should either derive their parameters or make them visibly tunable**. Hidden magic is a source of bugs and confusion for years.

## Related Notes

- [[MLFQ]] — textbook example.
- [[OS Design Goals]].
- [[Mechanism vs Policy]] — policy often contains voodoo constants; mechanism less so.
- [[Ch 8 — Scheduling - MLFQ]].
