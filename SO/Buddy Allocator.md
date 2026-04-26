---
title: Buddy Allocator
tags:
  - concept
  - ostep
  - memory
  - allocation
aliases:
  - Binary Buddy Allocator
  - Buddy Allocation
type: concept
introduced_in: Ch 17
---

# Buddy Allocator

> [!abstract] Definition
> The **buddy allocator** (Knowlton, 1965) keeps free memory in power-of-two-sized blocks. To allocate, recursively halve the smallest block that fits the request. To free, check whether the **buddy** (the other half from the last split) is also free; if so, coalesce upward, recursively. Used pervasively in the Linux kernel for physical-page allocation.

## The Tree

```
       ┌────── 64 KB ──────┐
       │                    │
       ├───────┬────────────┤
       │ 32 KB │ 32 KB      │
       ├───┬───┤
       │16K│16K│
       ├─┬─┤
       │8K│8K│ ← allocated 8 KB, leftmost
```

To allocate 7 KB:
1. Round up to 8 KB (smallest power of two that fits).
2. Find the smallest free block ≥ 8 KB.
3. If larger, split repeatedly until you reach an 8 KB block.

To free that 8 KB block:
1. Check whether its **buddy** (the other 8 KB at the same level) is also free.
2. If yes, merge into a 16 KB block.
3. Check the merged block's buddy at level 16 KB. Merge if free. Recurse.
4. Stop when a buddy is in use, or when you've merged all the way to the root.

## Why It's Clever

> [!tip] The buddy of address `A` at level `L` is `A XOR (1 << L)`
> One bit flip identifies the merge partner. Splitting and coalescing are essentially **bit manipulation**, not list traversals.

This makes coalescing **fast**, which is the whole point — the dominant cost in many allocators is coalescing decisions, and buddy turns it into a couple of arithmetic ops.

## Trade-Offs

| Pro | Con |
|---|---|
| Cheap coalescing (one bit flip) | Internal fragmentation: a 9 KB request consumes 16 KB |
| Bounded fragmentation | Power-of-two sizes only |
| Simple to implement | Wastes up to 50% within a block |

## Where It's Used

- **Linux kernel** — physical-page frame allocator. The "high half" of the kernel's [[Free-Space Management|free-space management]].
- **Various embedded RTOSes** — bounded fragmentation matters more than tightness.
- **Rarely** in user-space `malloc` — internal fragmentation is too costly for general C programs.

The Linux **slab allocator** sits on top of the buddy allocator: the buddy hands out pages, the slab carves them into objects of a fixed kind ([[Slab Allocator|see slab]]).

## Related Notes

- [[Slab Allocator]] — typically layered on top of buddy.
- [[Free-Space Management]] — the umbrella.
- [[Splitting and Coalescing]] — the operations buddy makes cheap.
- [[Internal Fragmentation]] — buddy's main downside.
- [[Allocation Strategy]] — buddy as an alternative to fit policies.
- [[Ch 17 — Free-Space Management]].
