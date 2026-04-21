---
title: Lottery Scheduling
tags:
  - concept
  - ostep
  - scheduling
  - algorithm
  - randomized
type: concept
introduced_in: Ch 9
---

# Lottery Scheduling

> [!abstract] Definition
> **Lottery scheduling** is a randomized [[Proportional Share Scheduling|proportional-share]] algorithm: each job holds a number of **tickets** representing its share, and every time slice the scheduler picks a random ticket; the holder gets the CPU. Over time, CPU shares converge on the ticket ratios.

Introduced by Waldspurger & Weihl (OSDI '94).

## Algorithm

```c
total = sum(tickets across all ready jobs)
winner = rand(0, total - 1)

counter = 0
for job in ready_list:
    counter += job.tickets
    if counter > winner:
        return job
```

Pick a random number in [0, total), walk the list summing tickets, the first job whose cumulative sum exceeds the winner is the scheduled one.

## Example

| Job | Tickets | Share |
|---|---:|---:|
| A | 100 | 25% |
| B | 50 | 12.5% |
| C | 250 | 62.5% |
| **Total** | **400** | **100%** |

A random ticket in [0, 400) selects one job; over many draws, C gets ~62.5% of slices.

## Why Randomness Is a Feature

> [!tip] Three properties of random approaches
> 1. **Avoids pathological workloads** — deterministic schedulers can have specific inputs that break them (e.g., LRU's worst-case cycles). Random has no worst case.
> 2. **Minimal state** — no per-process accounting beyond a ticket count. Compare with stride's pass values or CFS's vruntime.
> 3. **Fast** — random draw + list walk. No sort, no balance.

## Ticket Mechanisms

### Ticket Currency

Users get a local ticket budget (e.g., 100 each). Each user allocates their tickets among their jobs. OS converts local tickets to a global scale so one user can't dominate by inflating locally.

### Ticket Transfer

A process can temporarily hand its tickets to another — useful in client/server:

```
client → server: "here, take my tickets so you finish my work faster"
```

When done, server returns them.

### Ticket Inflation

In **trusted** environments, a process can temporarily raise/lower its own ticket count. Only works when processes don't compete adversarially — a malicious process would just inflate to infinity.

## Convergence to Fair Share

Short-term fairness: poor. Over 10 time slices, lottery can visibly deviate from the ticket ratio.

Long-term fairness: excellent. Over thousands of slices, lottery approximates the ratios arbitrarily well. Formally, the **Fairness metric** `F = T_first / T_second` (defined in OSTEP §9.4) approaches 1 as job length grows.

```
F=1.0 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
      ╱
F=0.5 ╱
      
      1   10   100   1000   (job length)
```

## Implementation — Pseudocode

```c
int counter = 0;
int winner = random(0, totaltickets);
node_t *curr = head;
while (curr) {
    counter += curr->tickets;
    if (counter > winner) break;
    curr = curr->next;
}
// curr is the winner; schedule it
```

Optimization: keep list sorted by ticket count **descending**, so the loop exits quickly on average.

## Handling I/O and Sleep

Jobs that block on I/O simply stop consuming lottery cycles. When they return, they rejoin the pool. This is **not ideal** — a frequently sleeping job loses its share while sleeping.

## Where It's Used Today

Lottery has been largely replaced by [[CFS - Completely Fair Scheduler|CFS]] (efficient) and variants for general-purpose use, but its ideas show up in:

- **VMware ESX** resource allocation.
- Research on **proportional-share** primitives in hypervisors.
- Teaching contexts (it's a great pedagogical example).

## Related Notes

- [[Proportional Share Scheduling]], [[Stride Scheduling]], [[CFS - Completely Fair Scheduler]].
- [[Scheduler]], [[Scheduling Metrics]].
- [[Ch 9 — Scheduling - Proportional Share]].
