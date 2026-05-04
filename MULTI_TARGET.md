# LEAP for Multi-Target / Cross-Language Projects

> How LEAP handles projects where the same specification must produce equivalent
> implementations in multiple programming languages from one source of truth.

## The central idea

Turn "works in language X" into "works in every language we target."

In a traditional cross-language port:

1. Reference implementation exists in language A
2. Someone reads the source, reimplements in language B
3. Subtle behavioral drift — different numeric edge cases, different string handling, different ordering
4. The two implementations diverge over time as A gets fixes B doesn't

The handoff is human-mediated and forks the truth. Bug fixes, new features, and
edge-case handling have to be applied N times by N humans.

LEAP inverts this:

1. The specification is written **language-neutral** — no Rust idioms, no C idioms, no Python idioms
2. One agent emits language A; another emits language B; another emits C
3. The same human-authored test suite validates every target
4. When a test fails in target B, the **spec** is fixed, then every target regenerates

**The spec is the only source of truth. The N implementations are byproducts.**

## Why this works now

Three things have quietly become production-ready:

1. **Cross-language reasoning by frontier LLMs** — the same model can emit idiomatic Rust, idiomatic C, and idiomatic Python from the same prose, given disciplined input
2. **Behavioral test corpora** — for established domains (parsers, file formats, protocols), the reference test suite is often the de-facto cross-language standard
3. **Byte-format and wire-format contracts** — many real systems have a frozen on-disk or on-wire spec; targeting that contract gives an unambiguous correctness oracle independent of any one language

What's new is treating the spec as the **primary artifact** — and explicitly forbidding it from leaking idioms of any single target language.

## Project structure

A multi-target LEAP project extends the base structure with a `parts/targets/`
directory for per-language mapping rules:

```
project-root/
├── master.md
├── schemas/                 # language-neutral contracts
│   ├── shapes.json          # abstract data shapes (JSON Schema)
│   └── opcodes.json         # state-machine / IR definitions
├── parts/
│   ├── parser/
│   │   ├── master.md        # behavioral spec, language-neutral
│   │   └── schema.json
│   ├── compiler/
│   │   └── ...
│   └── targets/             # ADDITIONAL — per-target mapping
│       ├── rust/
│       │   └── mapping.md   # how shapes/opcodes map to Rust idioms
│       ├── c/
│       │   └── mapping.md
│       ├── zig/
│       │   └── mapping.md
│       ├── go/
│       │   └── mapping.md
│       └── python/
│           └── mapping.md
├── tests/                   # one test corpus, runs against every target
├── src-rust/                # gitignored
├── src-c/                   # gitignored
├── src-zig/                 # gitignored
├── src-go/                  # gitignored
└── src-python/              # gitignored
```

### The `parts/targets/` directory

The UI equivalent of `design/`. It is the source of truth for *how* each target
language renders the language-neutral spec.

**Required files per target:**
- `mapping.md` — how schema shapes map to that language's idioms (e.g., "tagged unions become Rust enums; in C, become tagged structs with a discriminant field")

**Recommended:**
- A toolchain pin at the top of `mapping.md`: compiler version, stdlib API surface relied on. Silent stdlib breaks across versions are the most common failure mode.

### `src-<lang>/` instead of `src/`

The base SPEC says generated code lives in `src/`. Multi-target projects use
`src-<lang>/` per target. All are gitignored. All must be self-contained per
their language's conventions.

## Language-neutrality discipline (the hardest rule)

Every file in `parts/` and `schemas/` must be language-neutral. Concretely:

- **No Rust idioms.** No `Result<T, E>`, no `Option<T>`, no lifetimes, no traits-as-nouns. Describe errors as named conditions; describe optionality as "present or absent."
- **No C idioms.** No raw pointer arithmetic in prose, no `void*`, no `malloc`/`free` mentions in the contract, no struct-field-offset-as-spec.
- **No Python idioms.** No "this is a list comprehension"; no `__init__` semantics in spec text.
- **Express data shapes as abstract records.** Use JSON Schema for schemas; use English plus pseudo-code for spec semantics.
- **Express control flow as state machines or pseudo-code**, not as functions with a specific calling convention.
- **Express errors as named conditions** ("raise `INVALID_OPCODE` if X") — each generator maps that to idiomatic error handling in its target.

When a spec change is needed, ask: *can both a Rust generator and a C generator
implement this without either language feeling forced?* If the answer is no, the
spec is leaking. **Fix the spec before fixing the generators.**

The first time an idiom leaks is the hardest failure mode to recover from. By
the time you notice (target B regenerates with a workaround), the spec has
shifted toward target A. Catch this early.

## Convergent invention is a spec-gap signal

When two or more target agents independently invent the same workaround for
a missing piece of the spec, treat that convergence as a **spec bug**, not a
target bug.

- If only one target invents X → it's a target-mapping issue; fix the mapping
- If two or more independently invent X → the spec is silent where it shouldn't be; promote X into the spec and regenerate every sibling

This is one of the most reliable bug-class reduction rules in multi-target LEAP.

## The proof: sqlite-leap

[**sqlite-leap**](https://github.com/safitudo/sqlite-leap) demonstrates this at
scale: SQLite reimplemented from a language-neutral specification into five
languages from one spec.

| | sqlite-leap | mainline SQLite |
|---|---|---|
| Source LOC (across 5 targets) | ~250,000 | ~150,000 (one language) |
| Spec LOC | ~28,000 | n/a (the code is the spec) |
| Targets | C, Rust, Zig, Go, Python | C |
| Upstream sqllogictest excl-SKIP | 98.88–99.98% per target | 100% |
| `exec/mainline` (denominator parity) | 99.93–99.97% on 4 compiled targets | 100% |
| Crashes on full corpus | 0 across 4 compiled targets | 0 |
| On-disk byte-identity at fixed fixtures | SHA1-identical, 5/5 + mainline | n/a |

Two fixed fixtures (a 270-row split file and a 5,000-row deep-split file)
produce SHA1-identical `.db` files across all five leap targets and mainline
SQLite, and mainline `PRAGMA integrity_check` returns `ok` on every leap-emitted
file.

Reproduction: see the
[v0.1.1 release tarballs](https://github.com/safitudo/sqlite-leap/releases/tag/v0.1.1-publication-2026-04-30)
and `bench/PUBLISHED.md` for raw CSV citations.

## What this unlocks

- **Cross-language libraries by default** — same spec, ship five language packages, every consumer gets the same behavior
- **Reference-implementation parity** — when behavior is governed by an external standard (file format, wire protocol), every target stays bit-compatible
- **Language-trade-off shopping** — pick the target that best fits a use case (smallest binary, fastest cold start, easiest packaging), without forking the truth
- **Compiler-vs-runtime experiments** — same spec, AOT-compiled C and JIT-friendly Python from one source

## The honest caveats

- **The spec is harder to write.** Single-language specs let you lean on the language's vocabulary; language-neutral specs force you to invent your own. Expect to throw away the first attempt.
- **Some leaves resist regeneration past a certain size.** Empirically we see ~3K LOC per leaf-part as the reliable agent regen envelope. Past ~15–20K LOC, regeneration stops being one-shot — sub-decompose the leaf or hand-maintain that file (and document the regen-debt).
- **Per-target performance equality is not free.** Same spec produces correctness parity, not speed parity. Performance characteristics still depend on the target's runtime model. Plan for performance work to be partly target-local even when correctness is fully spec-driven.
- **Toolchain breaks are the silent killer.** Pin the compiler/stdlib version per target in `mapping.md` and verify it on every regeneration. We've had a stdlib API change silently break 5/5 targets simultaneously.
- **Reputation asymmetry is real.** A multi-target reimplementation of an established system inherits the original's reputation only after independent ecosystem validation. Ship as compatibility-tier first.

## Getting started

1. Read [SPEC.md](./SPEC.md) for the base LEAP structure
2. Add `parts/targets/<lang>/mapping.md` for each language you target, naming the toolchain version
3. Audit your existing `parts/` for idiom leaks before adding the second target
4. Add target-agnostic test runners that can validate the same behavior against any `src-<lang>/`

This methodology is at v0.3-draft. The proof exists ([sqlite-leap](https://github.com/safitudo/sqlite-leap)). The methodology document is being extended in lockstep.

---

**Status:** Draft methodology v0.3. Validated by sqlite-leap (5 targets, 250K LOC, byte-identity, full upstream test corpus). Open to extension as more multi-target LEAP projects ship.
