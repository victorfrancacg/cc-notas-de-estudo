---
title: Space Sharing
tags:
  - concept
  - ostep
type: concept
introduced_in: Ch 2
---

# Space Sharing

> [!abstract] Definition
> **Space sharing** is the strategy of letting many actors share one resource by **dividing the resource into pieces**, each assigned to a different actor.

## Canonical Examples

| Resource | Space-sharing unit |
|---|---|
| Disk | **Blocks** — each [[File]] owns a distinct set of disk blocks. |
| DRAM | **Pages** — each process owns its own physical pages; no two processes share the same page (except when deliberately sharing). |
| Address space | **Regions** — stack, heap, code, data occupy disjoint ranges. |

## When Space Sharing Works Well

- The resource can be **cleanly partitioned** with low overhead (e.g., disk blocks are naturally discrete).
- Sharing is **long-lived** — once allocated, the partition stays assigned for a while (unlike CPU time, where actors swap constantly).
- **Concurrent access** would cause correctness problems if the resource were interleaved in time.

## When It Doesn't

- Highly contested, time-slicing resources (like the CPU) would waste too many pieces if space-shared.
- Extremely imbalanced demand — one actor wants 99%, others want a trickle — leads to internal fragmentation.

## Contrast with Time Sharing

See [[Time Sharing]]. A quick mnemonic:

> [!tip] Split the *axis* that's cheapest to divide
> - If time is cheap to slice (microsecond context switches): **time sharing**.
> - If space is cheap to slice (contiguous blocks, discrete pages): **space sharing**.

## Related Notes

- [[Time Sharing]] — the opposite strategy.
- [[Virtualization]] — both are tools for virtualizing a shared resource.
- [[File System]], [[Paging]] — classic users of space sharing.
