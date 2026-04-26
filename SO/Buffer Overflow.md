---
title: Buffer Overflow
tags:
  - concept
  - ostep
  - memory
  - bug
  - security
aliases:
  - Buffer Overrun
  - Stack Smashing
type: concept
introduced_in: Ch 14
---

# Buffer Overflow

> [!abstract] Definition
> A **buffer overflow** is a write past the end of an allocated buffer, corrupting whatever memory follows. Famous as the foundational technique for many security exploits, but also a routine bug that destabilizes programs without any malice involved.

## The Canonical Bug

```c
char *src = "hello";
char *dst = malloc(strlen(src));   // ❌ forgot the +1 for '\0'
strcpy(dst, src);                  // writes 'h','e','l','l','o','\0'
                                   // → 1 byte past the allocation
```

The single overrun byte may:

- Land in the allocator's next-block header → corrupts the heap (visible later).
- Land in unrelated user data → silent corruption.
- Land in a guard page → clean [[Segmentation Fault|segfault]].
- Trigger no symptom at all if the byte was unused padding — until a different test hits the wrong layout.

## Stack Buffer Overflow → Control-Flow Hijack

```c
void f(char *src) {
    char dst[8];
    strcpy(dst, src);    // src longer than 8 bytes → overruns
}
```

A stack frame typically lays out as:

```
[ caller's data ]
[ saved frame pointer ]
[ saved return address ]   ← !
[ local variables       ]
[ dst[8]                ]
```

Overrunning `dst` upward overwrites the **saved return address**. The attacker chooses bytes that point to attacker-controlled code (or, with NX bit + ASLR, to existing code chosen via ROP). The function returns to the wrong address, executing the attacker's instructions.

> [!warning] This is the original "stack smashing" exploit
> Aleph One's 1996 *Smashing The Stack For Fun And Profit* (`Phrack #49`) — required reading for the security mindset.

## Defenses

| Defense | What it does |
|---|---|
| **Bounded string functions** (`strncpy`, `snprintf`, `strlcpy`) | Take a max length |
| **Stack canaries** (`-fstack-protector`) | Guard word before saved RA; checked at function exit |
| **NX bit / DEP / W^X** | Stack pages non-executable — shellcode on stack won't run |
| **ASLR** | Randomize stack/heap/library base addresses — exploits can't hardcode targets |
| **AddressSanitizer** | Runtime detection; pads buffers with poison bytes |
| **Fuzzing** | Drive code with random inputs to surface overflows |
| **Safer languages** | Rust, Go, Java — bounds checks built in |

## Buffer Overflow vs Heap Buffer Overflow

- **Stack overflow** → corrupts saved return address, frame pointer, locals.
- **Heap overflow** → corrupts allocator metadata (next-block header) → on the next `free`/`malloc`, attacker controls a free-list write → arbitrary write primitive.

Both are exploitable; heap overflows are typically harder to weaponize but very real (UAF + heap-spray, type confusion, etc.).

## Related Notes

- [[Stack (Runtime)]], [[Heap (Runtime)]] — the two arenas overflows occur in.
- [[Segmentation Fault]] — the benign symptom when the OS catches it.
- [[Memory Leak]], [[Dangling Pointer]], [[Double Free]] — the bug-family neighbors.
- [[Isolation (OS)]] — what overflows attempt to subvert.
- [[Ch 14 — Interlude - Memory API]].
