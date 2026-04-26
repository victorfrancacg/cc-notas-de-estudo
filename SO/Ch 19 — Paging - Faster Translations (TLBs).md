---
title: Ch 19 — Paging - Faster Translations (TLBs)
tags:
  - ostep
  - chapter
  - part/virtualization
  - memory
  - cache
type: chapter
book: OSTEP
chapter: 19
pages: 213-228
---

# Ch 19 — Paging: Faster Translations (TLBs)

> [!abstract] One-sentence summary
> The [[TLB]] (Translation Lookaside Buffer) is a small, fast hardware cache of recent virtual-to-physical translations, sitting inside the [[MMU]]. With a high hit rate it makes [[Paging|paging]] fast enough to be invisible. Misses fall through to a full [[Page Table]] walk — handled either by hardware (x86) or by an OS trap handler (MIPS, RISC). Context switches force the OS to either flush the TLB or tag entries with **[[ASID|ASIDs]]**.

## Crux

> [!example] The Crux: How To Speed Up Address Translation
> How can we speed up address translation, and generally avoid the extra memory reference that paging seems to require? What hardware support is required? What OS involvement is needed?

## 19.1 TLB Basic Algorithm

The MMU consults the TLB *before* any page table walk:

```c
VPN = (VA & VPN_MASK) >> SHIFT;
(hit, entry) = TLB_Lookup(VPN);

if (hit) {                                 // TLB hit
    if (!CanAccess(entry.ProtectBits)) raise(PROT_FAULT);
    PA = (entry.PFN << SHIFT) | offset;
    Register = AccessMemory(PA);
} else {                                   // TLB miss
    PTE = AccessMemory(PTBR + VPN * sizeof(PTE));   // ← extra access
    if (!PTE.Valid) raise(SEGFAULT);
    if (!CanAccess(PTE.ProtectBits)) raise(PROT_FAULT);
    TLB_Insert(VPN, PTE.PFN, PTE.ProtectBits);
    RetryInstruction();
}
```

A **hit** costs essentially nothing. A **miss** costs one extra memory access (or several with [[Multi-Level Page Table|multi-level tables]]).

## 19.2 Hit Rate via Spatial Locality

Walk an array `a[0..9]`, 10 elements, 16-byte page, 4-byte ints. Three array elements per page → first access on each page misses, remaining hit:

```
miss, hit, hit, miss, hit, hit, hit, miss, hit, hit
```

**Hit rate = 70%** even on the first pass — driven entirely by [[Spatial Locality]]: once page `P` is in the TLB, the next access on `P` is free.

A second pass would hit ~100% (assuming the TLB holds those 3 entries) — that's [[Temporal Locality]].

> [!tip] TLBs work because programs are local
> Programs touch a few pages a lot, then move on. The TLB exploits this — same as every other cache in the memory hierarchy.

## 19.3 Who Handles a TLB Miss?

### Hardware-Managed TLB (CISC: x86)

The hardware itself walks the page table on a miss:

- Knows the page-table format (encoded in silicon).
- Knows where the table is (`CR3`).
- Refills the TLB and retries the instruction — invisible to the OS.

### Software-Managed TLB (RISC: MIPS, SPARC)

On a miss, the hardware just **traps** to an OS-supplied handler:

```text
TLB miss → trap → OS handler runs:
    PTE = lookup(VPN);          // OS code can use any data structure
    TLB_Insert(VPN, PTE.PFN);
    return-from-trap            // hardware retries the instruction
```

Pros and cons:

| Hardware-managed | Software-managed |
|---|---|
| One PTE format, fixed in silicon | OS picks any data structure |
| Faster on miss | More flexible (e.g., hash tables, multi-level) |
| Used by x86 family | Used by MIPS/SPARC/older Alpha |

> [!warning] Don't infinite-loop the miss handler
> The handler itself is in virtual memory. If accessing it causes another TLB miss, you have an infinite chain. Solutions: keep handler code unmapped (physical addresses only), or keep its translations **wired** (permanently in TLB).

## 19.4 What's in a TLB Entry

A TLB is a **fully-associative cache**: any entry can be anywhere; lookup checks all entries in parallel. Each entry holds:

```
+-----+-----+-------+----------+-------+-------+
| VPN | PFN | valid | protect  | dirty | ASID  |
+-----+-----+-------+----------+-------+-------+
```

> [!info] TLB valid bit ≠ PTE valid bit
> - PTE valid = "this VPN is mapped at all" (otherwise → segfault).
> - TLB valid = "this entry contains a translation right now" (otherwise → it's empty/clearable). Used to flush the TLB on context switch.

## 19.5 Context Switches — The Big TLB Problem

The TLB caches translations for the **currently running process**. After a [[Context Switch]], those translations are *wrong*:

```
Before switch (P1 was running):
  VPN 10 → PFN 100, valid, ASID=1

After switch to P2 (which has VPN 10 → PFN 170):
  Old entry still says VPN 10 → PFN 100 ← bug!
```

### Solution 1: Flush

On every context switch, mark all TLB entries invalid. Simple. **Costly** — the next process starts cold.

### Solution 2: [[ASID|Address Space Identifiers]]

Tag each TLB entry with the ASID of the process it belongs to:

| VPN | PFN | valid | ASID |
|---|---|---|---|
| 10 | 100 | 1 | 1 |
| 10 | 170 | 1 | 2 |

The MMU compares the current process's ASID (set by the OS in a special register) with the entry's ASID. Mismatch → treat as miss. With ASIDs, the TLB doesn't need to be flushed; entries from multiple processes coexist.

x86 calls this PCID (Process-Context Identifier); ARM has ASID. 8-bit ASIDs are common — meaning when more than 256 processes run, the OS recycles ASIDs and is careful to flush stale entries.

## 19.6 Replacement Policy

When the TLB is full and a new entry needs to land, which old one gets evicted?

- **LRU** (Least Recently Used) — typically wins; matches locality.
- **Random** — surprisingly competitive; resists pathological workloads (a loop touching one more page than the TLB holds → LRU thrashes 100%, random does much better).

Real TLBs use approximations of LRU (NRU, clock-like) because true LRU is expensive in hardware.

## 19.7 A Real TLB Entry — MIPS R4000

```
+----+--------+--------+-----+-----+----+----+----+
|    |  VPN   |   G    |     | ASID  |    |    |   |
+----+--------+--------+-----+-----+----+----+----+
|    |       PFN       |     |     | C  | D  | V |
+----+-----------------+-----+-----+----+----+----+
```

- **VPN** (19 bits — half the address space is reserved for kernel).
- **PFN** (24 bits → 64 GB physical RAM).
- **G** (global) — if set, ignore ASID (kernel pages used by all processes).
- **ASID** (8 bits).
- **C** (coherence) — caching attributes.
- **D** (dirty) — actually a writable bit, not "dirty" in the usual sense.
- **V** (valid).

MIPS uses **software-managed TLB** with privileged instructions (`TLBP`, `TLBR`, `TLBWI`, `TLBWR`). Some entries can be **wired** — permanently resident — for handler code.

## 19.8 Issues: TLB Coverage, Physical vs Virtual Caches

> [!warning] TLB coverage
> A 64-entry TLB with 4 KB pages covers only 256 KB of working set. Programs with hot data >256 KB miss the TLB constantly. Mitigation: **huge pages** (2 MB → 64 entries cover 128 MB).

> [!warning] Physically-indexed cache penalty
> If the L1 cache is physically indexed, address translation must complete *before* cache lookup — adding latency to the critical path. CPUs use clever tricks (virtually-indexed physically-tagged, VIPT) to overlap translation with cache access.

## 19.9 Summary

> [!example] Crux answered
> The TLB makes paging viable. On the common path (TLB hit) translation is essentially free; misses fall through to either hardware-walked or OS-walked page tables. Context switches force flushing or ASID tagging. *"RAM isn't always RAM"* — Culler's Law: when your working set exceeds TLB coverage, accesses get expensive.

## Aside: Use Caching When Possible

> [!tip] The most fundamental performance technique
> Caches exploit **[[Spatial Locality|spatial]]** and **[[Temporal Locality|temporal]] locality**. The TLB is just one in a hierarchy: L1d, L1i, L2, L3, RAM, disk, network. The whole point of "fast in the common case" is having a smaller cache nearer the CPU than where the data really lives.

## Related Notes

- [[TLB]] — the standalone concept, deep dive.
- [[Page Table]] — what the TLB caches entries from.
- [[ASID]] — process-tagged TLB entries.
- [[Spatial Locality]], [[Temporal Locality]] — why caches work.
- [[Hardware-Managed vs Software-Managed TLB]] — CISC vs RISC TLB miss handling.
- [[Context Switch]] — the moment the TLB becomes a problem.
- [[MMU]] — the TLB lives inside it.
- [[Paging]] — the technique TLBs make fast.
- [[Page Fault]] — what happens beneath TLB miss when page isn't resident.
- [[Ch 18 — Paging - Introduction]], [[Ch 20 — Paging - Smaller Tables]].
- Index: [[MOC - Memory Virtualization]].
