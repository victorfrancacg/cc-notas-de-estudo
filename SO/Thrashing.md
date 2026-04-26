---
title: Thrashing
tags:
  - concept
  - ostep
  - memory
  - performance
type: concept
introduced_in: Ch 22
---

# Thrashing

> [!abstract] Definition
> **Thrashing** is the pathological state in which a system spends most of its time servicing [[Page Fault|page faults]] (paging in/out) rather than running user code. Symptoms: very high disk I/O, near-zero CPU utilization, unresponsive UI. Caused by the combined [[Working Set|working set]] of running processes exceeding physical memory.

## How It Happens

```
1. System runs many processes; total WSS > RAM.
2. Each access has high P_Miss; faults dominate.
3. Each fault evicts another active page → cascading faults.
4. CPU sits waiting for disk; user-visible work stalls.
5. New processes arriving make it worse.
```

The system is technically progressing, but at disk-paging speeds — 10⁴–10⁵× slower than steady-state.

## Recognizing Thrashing

| Symptom | What you see |
|---|---|
| CPU utilization | drops to ~0% (waiting for disk) |
| Disk I/O | pegged at 100% |
| Page faults/sec | extremely high (Linux: `vmstat` shows `si`/`so` columns spike) |
| User experience | UI freezes, terminal latency, "fans spin up" |
| Memory pressure metrics | Linux `psi` — pressure-stall information — shows mem stall ≫ 0 |

## Why The Vicious Cycle

- Process A faults on page P → evict B's page Q.
- Process B faults on page Q → evict A's page R.
- Each process "steals" from the others, in pairs and triples.
- All processes' working sets shrink below their actual needs.
- Faults beget faults.

## Mitigations

### Working-Set Admission Control

Refuse to schedule new processes when their addition would push aggregate WSS beyond RAM. Older OSes (notably VAX/VMS) did this. Linux mostly doesn't.

### Out-of-Memory (OOM) Killer

Linux's pragmatic approach: when memory is exhausted, the kernel **kills a memory-hogging process**. Heuristic chooses the victim based on memory usage, runtime, and `oom_score_adj`. Crude — sometimes the wrong process dies — but prevents system-wide collapse.

### Compression / zswap / zram

Compress evicted pages in RAM instead of writing to disk. Trades CPU for I/O — typically a win on modern hardware.

### Cgroups Memory Limits

Linux cgroups enforce per-group memory quotas; OOM kicks in within a group, isolating thrashing to the offending workload.

### Buy More RAM

The simplest, often-best fix. Decades of cheap DRAM have made thrashing relatively rare on well-provisioned systems.

## A Brief History

Thrashing was a critical concern in early time-sharing systems (1960s-70s) when RAM was expensive and oversubscription was the norm. Denning's [[Working Set|working-set model]] formalized the diagnosis. Today, with abundant RAM, thrashing is mostly a misconfiguration symptom (memory leak, undersized container, runaway process).

## Related Notes

- [[Working Set]] — the model that predicts thrashing.
- [[Page Fault]] — the costly event thrashing maximizes.
- [[Page Replacement]] — even an optimal policy can't save you when WS > RAM.
- [[Swap Space]] — the disk thrashing hammers.
- [[Ch 22 — Beyond Physical Memory - Policies]].
