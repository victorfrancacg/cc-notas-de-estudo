---
title: Ch 8 — Scheduling - MLFQ
tags:
  - ostep
  - chapter
  - part/virtualization
  - scheduling
type: chapter
book: OSTEP
chapter: 8
pages: 87-98
---

# Ch 8 — Scheduling: The Multi-Level Feedback Queue

> [!abstract] One-sentence summary
> **[[MLFQ]]** is a scheduler that approximates [[SJF Scheduling|SJF]] and [[Round Robin]] simultaneously by **learning from job behavior**: jobs start at high priority, get demoted when they use their time budget, and get boosted periodically to prevent starvation.

## Crux of the Problem

> [!example] How to Schedule Without Perfect Knowledge?
> How can we design a scheduler that both **minimizes response time** for interactive jobs and **minimizes turnaround time** — without knowing job lengths in advance?

## Historical Note

MLFQ was invented in **1962** by Corbató et al. for the Compatible Time-Sharing System (CTSS), and refined on Multics. Corbató received the **Turing Award** partly for this work.

Real modern schedulers using MLFQ or close variants: BSD UNIX, Solaris, Windows NT lineage.

## The Key Insight

> [!tip] Learn from history
> Jobs have **phases of behavior** — sometimes CPU-bound, sometimes I/O-bound / interactive. If we watch what a job *just did*, we can predict what it's likely to do next and schedule accordingly.

## The Final Five Rules

See [[MLFQ]] for the full treatment. Summary:

1. **Rule 1:** If Priority(A) > Priority(B), A runs.
2. **Rule 2:** If Priority(A) = Priority(B), A and B share via [[Round Robin]].
3. **Rule 3:** When a job enters, it starts at the **highest priority**.
4. **Rule 4:** If a job uses up its total time **allotment** at a level (no matter how many CPU bursts), its priority is **reduced** (demotion).
5. **Rule 5:** Every period *S*, **boost** all jobs to the top queue.

## The Four Attempts That Got Us Here

| Attempt | Added | Problem it solved | Problem it left |
|---|---|---|---|
| #1 | Demote on full time-slice use | Prevent long jobs from hogging high priority | Starvation, gaming, behavior changes |
| #2 | [[Ousterhout's Law|Periodic boost]] (Rule 5) | Starvation, behavior changes | Gaming |
| #3 | Better accounting (Rule 4 tracks cumulative CPU, not per-burst) | Gaming | Tuning the magic numbers |

## Why MLFQ Approximates SJF

Short interactive jobs enter at the top, finish quickly while still at high priority → behave like SJF for the short case. Long jobs gradually sink to lower queues → runs there, not blocking the interactive ones. Overall turnaround + response → *both* improve.

## The Gaming Attack

> [!warning] Malicious use of Rule 4a/4b
> If Rule 4 said "stay at level if you relinquish before allotment expires", a clever process could issue a tiny I/O right before expiry, stay at top forever, and monopolize 99% of CPU.
>
> **Fix:** cumulative accounting. Once you've burned up your total allotment at a level — whether in one burst or ten — you get demoted.

See [[Gaming the Scheduler]].

## Voodoo Constants

> [!tip] [[Ousterhout's Law]]
> The period *S* for the priority boost is a classic "voodoo constant" — nobody can pick it with justification. Too high: long jobs starve. Too low: interactive fairness erodes.
>
> "Avoid voodoo constants where possible. When you can't, document them and allow tuning."

## Other Features Real MLFQs Have

- **Variable time slice by queue** — top queues (interactive) get short slices (~10 ms); bottom queues (CPU-bound) get long ones (~100 ms). Amortizes switch cost better for long-running jobs.
- **OS-reserved top queues** — user jobs can't reach the highest priorities.
- **User advice** — `nice` value (UNIX) lets users hint at priority.
- **Decay-usage formulas** — FreeBSD uses math instead of discrete queues.

## Tip: Use Advice Where Possible

Users and admins often know things the OS can't infer. Interfaces to *provide hints* (like `nice`, `madvise`, `posix_fadvise`) let the OS make better decisions without needing omniscience.

## Related Notes

- [[MLFQ]] — the full algorithm and rules.
- [[Gaming the Scheduler]] — the class of exploit MLFQ had to fix.
- [[Starvation (Scheduling)]] — what priority boost cures.
- [[Ousterhout's Law]] — on voodoo constants.
- [[Scheduler]], [[Scheduling Metrics]].
- [[SJF Scheduling]], [[Round Robin]] — what MLFQ synthesizes.
- [[Ch 7 — Scheduling - Introduction]] — the baseline.
- [[Ch 9 — Scheduling - Proportional Share]] — a different family of schedulers.
