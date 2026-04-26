---
title: Reference Bit
tags:
  - concept
  - ostep
  - memory
  - paging
  - hardware
aliases:
  - Use Bit
  - Accessed Bit
  - A bit
type: concept
introduced_in: Ch 22
---

# Reference Bit (Use Bit)

> [!abstract] Definition
> The **reference bit** (a.k.a. **use bit**, **accessed bit**, `A` bit on x86) is a single bit per [[Page Table Entry|PTE]] that the hardware **automatically sets to 1** whenever the corresponding [[Page|page]] is accessed. The OS clears it back to 0. Provides cheap hardware support for approximating [[LRU]] in [[Page Replacement|replacement policies]] like [[Clock Algorithm|Clock]].

## How It Works

Three steps repeat indefinitely:

1. **Hardware**: on any load/store/instruction-fetch, set `PTE.A = 1` via the [[MMU]] (no OS involvement, no perceptible cost).
2. **OS** (periodically or on eviction sweep): reads the bits, clears them.
3. **OS** (on eviction): pages with `A = 0` haven't been touched since the last clear → good eviction candidates.

> [!info] One bit summarizes "has been touched at least once since last clear"
> Not strict LRU, but a useful approximation: pages with `A=0` were definitely cold during the recent sweep window.

## Why a Single Bit, Not a Counter

Counters or timestamps would give finer-grained LRU but cost:

- Wider PTEs.
- Hardware updates every access (a write to memory or a buffered counter).
- More OS scan work to compare counters.

A 1-bit signal is essentially **free** for the hardware to maintain and good enough to drive Clock.

## When the OS Clears the Bit

Two strategies:

- **On eviction sweep** — the [[Clock Algorithm|clock hand]] clears `A=1` bits as it passes, looking for `A=0`. Implicit clearing.
- **Periodic sweep** — a kernel thread (e.g., `kswapd` in Linux) walks pages every N seconds, clearing bits, observing which got reset before next sweep.

Linux uses both: kswapd for proactive scanning, plus reclaim-time sweeping when memory pressure spikes.

## What If Hardware Doesn't Provide One?

Some early systems (notably **VAX**) lacked a reference bit. The OS emulated it using **protection bits** (Babaoglu & Joy, 1981):

1. Mark a page **inaccessible** (clear all R/W permissions).
2. On access, hardware traps to OS.
3. OS notes "this page is referenced", restores permissions, returns.
4. Periodically, mark pages inaccessible again to retest.

Effective, but adds trap overhead per first-access. Modern hardware all provides reference bits.

## Reference Bit ≠ Dirty Bit

| Bit | Set when |
|---|---|
| Reference (`A`) | any access — read, write, exec |
| Dirty (`D`) | **write** only |

Both are auto-set by hardware, cleared by OS. Replacement policies use them together: prefer `(A=0, D=0)` victims (cold + clean — no I/O on eviction).

## Related Notes

- [[Page Table Entry]] — where the bit lives.
- [[Clock Algorithm]] — the canonical user.
- [[LRU]] — what reference bits approximate.
- [[Page Replacement]] — the umbrella decision.
- [[MMU]] — the hardware that sets the bit.
- [[Ch 22 — Beyond Physical Memory - Policies]].
