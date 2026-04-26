---
title: Ch 16 — Segmentation
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
type: chapter
book: OSTEP
chapter: 16
pages: 169-180
---

# Ch 16 — Segmentation

> [!abstract] One-sentence summary
> **Segmentation** generalizes [[Base and Bounds]]: instead of one base/bounds pair for the whole [[Address Space]], have *one per logical segment* (code, heap, stack). This eliminates the [[Internal Fragmentation|internal fragmentation]] of base/bounds, supports **sparse address spaces**, and enables **code sharing** — but introduces **[[External Fragmentation|external fragmentation]]** as a new problem.

## Crux

> [!example] The Crux: How To Support A Large Address Space
> How do we support a large address space with (potentially) lots of free space between the stack and the heap? With small pretend address spaces base/bounds is fine, but a 32-bit (4 GB) address space using only a few MB demands 4 GB of physical residency. Unsustainable.

## 16.1 Segmentation: Generalized Base/Bounds

> [!info] One base/bounds *per segment*
> Have a separate base/bounds pair for each logical region — typically code, heap, stack. Each can be placed independently in physical memory. The unused gap between heap and stack is **never allocated** in physical memory at all.

```
Virtual Address Space               Physical Memory
   0KB  ┌──────────┐                 ┌──────────────┐ 0KB
        │ Code     │                 │ Operating    │
        │ Heap     │                 │ System       │
        │   ↓      │           16KB ┌├──────────────┤
        │ (free)   │                 │ Stack        │ 28K base, 2K size
        │   ↑      │           28KB └│              │ (grows down)
        │ Stack    │           32KB ┌│ Code         │ 32K base, 2K size
  16KB  └──────────┘           34KB │├──────────────┤
                               34KB ┌│ Heap         │ 34K base, 3K size
                               37KB └└──────────────┘
```

Three segments → three base/bounds register pairs. Physical residency only for the actually-used parts.

## 16.2 Which Segment Are We Referring To?

The hardware needs to know which segment a virtual address belongs to.

### Explicit approach — top bits

Reserve the high bits of the virtual address as a **segment selector**:

```
13 12 11 10 ... 1 0
[ seg ][   offset   ]
  2 bits   12 bits
```

`seg=00` → code, `seg=01` → heap, `seg=11` → stack. The hardware uses the selector to pick the right base/bounds pair, applies them to the offset.

### Implicit approach

The hardware **infers** the segment from how the address was generated:

- From the [[Program Counter]] → code segment.
- From the stack pointer / frame pointer → stack segment.
- Otherwise → heap.

Saves bits in the address but is a bit magical.

## 16.3 The Stack Grows Negative

The stack is special: it grows toward *lower* addresses. The segment register set therefore needs an extra bit indicating direction:

| Segment | Base | Size | Grows + ? |
|---|---|---|---|
| Code | 32 K | 2 K | 1 |
| Heap | 34 K | 3 K | 1 |
| Stack | 28 K | 2 K | 0 |

Translation logic for negative-growth: subtract the maximum segment size from the offset to get a negative value, then add the (top-of-segment) base.

## 16.4 Support for Sharing — Protection Bits

Add per-segment **protection bits** (read / write / execute):

| Segment | Protection |
|---|---|
| Code | Read-Execute |
| Heap | Read-Write |
| Stack | Read-Write |

A read-only code segment can be **shared** across multiple processes mapping it: each process thinks it has its own private code, but physically only one copy exists. Memory savings + the [[Isolation (OS)|isolation]] guarantee remains because the segment is read-only.

## 16.5 Fine-Grained vs Coarse-Grained Segmentation

- **Coarse-grained** — a few segments (code, heap, stack). The OSTEP examples. MMU has a few base/bounds pairs.
- **Fine-grained** — thousands of small segments (Multics, Burroughs B5000). Requires a **segment table** stored in memory. More flexible (objects-as-segments), but more overhead.

## 16.6 OS Issues

Segmentation hands the OS several new problems:

### [[Context Switch]]
Save and restore *all* segment registers (base/bounds/grow-direction/protection per segment) on every switch.

### Growing/shrinking segments
A `malloc` may need to grow the heap segment. The OS tries to enlarge in place; if no room, it must move the segment elsewhere in physical memory and update the heap base.

### [[External Fragmentation]] — the killer downside
Variable-sized segments lead to physical memory looking like swiss cheese:

```
Not Compacted                   Compacted
0   ┌─────────┐ OS              0  ┌─────────┐ OS
16  ├─────────┤
24  │ alloc   │                 24 │ alloc   │
32  ├─────────┤                    │         │
40  │ alloc   │                 40 │ free    │
48  ├─────────┤
56  │ alloc   │                 56 ├─────────┤
64  └─────────┘                 64 └─────────┘
   (24K free in 3 holes)         (24K free, contiguous)
```

A 20 KB request fails in the left layout despite 24 KB free, because no *contiguous* 20 KB chunk exists.

**Fixes:**
- **Compaction** — periodically rearrange segments to coalesce free space. Expensive (memory-bandwidth-bound).
- **Smarter free-list policies** — best-fit, worst-fit, first-fit, buddy. None eliminates the problem; see [[Free-Space Management|Ch 17]].

> [!warning] Tip: "If 1000 solutions exist, no great one does"
> The fact that allocators have a hundred-paper history of trying to minimize external fragmentation hints there is no clean fix. The next chapter ([[Paging]], Ch 18) will sidestep the problem entirely by allocating in **fixed-size** units.

## 16.7 Summary

> [!example] Crux answered
> Segmentation supports large, sparse address spaces by giving each logical region its own base/bounds pair, and enables read-only code sharing across processes via protection bits. Its weakness — variable-sized chunks producing **external fragmentation** — motivates the next chapter ([[Free-Space Management]]) and ultimately the move to fixed-size [[Paging]].

## Aside: The Origin of "Segmentation Fault"

> [!info] Why your C bugs say "segfault"
> The term comes directly from this chapter: an out-of-bounds reference into a segment is a **segmentation violation**. The signal is named after the era; modern hardware uses paging, not segments, but the name `SIGSEGV` persists.

## Related Notes

- [[Segmentation]] — the standalone concept note.
- [[Base and Bounds]] — what segmentation generalizes.
- [[External Fragmentation]] — the new problem segmentation creates.
- [[Internal Fragmentation]] — what it eliminates.
- [[Sparse Address Space]] — what segmentation supports.
- [[Memory Compaction]] — one fix for external fragmentation.
- [[Free-Space Management]] — Ch 17, allocators that mitigate the problem.
- [[Paging]] — Ch 18, the alternative that eliminates external fragmentation.
- [[Segmentation Fault]] — the signal named after this mechanism.
- Index: [[MOC - Memory Virtualization]].
