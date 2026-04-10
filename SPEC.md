# LEAP Project Specification

This document defines the canonical structure of a LEAP project. Any project following this layout is LEAP-compatible and can be built by any LEAP-aware agent.

## Directory structure

```
project-root/
├── master.md           # REQUIRED — root orchestration prompt
├── schemas/            # REQUIRED — shared contracts between parts
│   └── *.ts            # TypeScript, JSON Schema, or other contract format
├── parts/              # REQUIRED — one folder per component
│   └── {name}/
│       ├── master.md   # Prompt for this part
│       ├── schema.ts   # Contract for this part
│       └── parts/      # OPTIONAL — recursive sub-parts
├── tests/              # REQUIRED — human-authored behavioral tests
├── src/                # REQUIRED in .gitignore — generated code
└── _original/          # OPTIONAL — reference code during rewrite mode only
```

## File roles

### `master.md` (root)

The root prompt. The entry point for any agent building the project.

**Must contain:**
- Description of what the application does
- Architecture overview (how parts connect)
- Tech constraints (language, runtime, dependencies)
- Generation rules (how to wire parts together)
- Verification steps (how to run tests)
- Running instructions (how to launch the app)

**Should open with** an imperative instruction to the agent: "Read all schemas and parts, generate code, run tests, fix until green."

### `schemas/`

Shared type definitions and contracts that multiple parts depend on.

**Format:** TypeScript interfaces (preferred), JSON Schema, or any machine-readable contract language.

**Purpose:** Define the architecture. Any module can reference these types without importing from another part's implementation.

### `parts/{name}/master.md`

Per-part prompt. Describes what this specific component does.

**Must contain:**
- Behavioral description (what the part must do)
- Edge case handling
- Output specification (which files to generate in `src/`)

### `parts/{name}/schema.ts`

Per-part contract. Defines the exact interface this component must expose.

**Rule:** Other parts may import from this schema file, never from the generated implementation.

### `tests/`

Human-authored tests. These are the specification of correctness.

**Rules:**
- Test behavior, not structure
- No snapshot tests that lock implementation details
- Tests should pass regardless of which AI generated the code
- Prefer integration tests over mocked unit tests

### `src/`

Generated code. **Must be in `.gitignore`.**

This is where an agent writes the code after reading the prompts, schemas, and tests.

### `_original/` (rewrite mode only)

If present, contains the reference implementation being rewritten. Used during rewrite mode for analysis and test extraction. **Must not be read during code generation.**

## Generation rules

A LEAP-compatible agent MUST:

1. Read `master.md` at the project root
2. Read all files in `schemas/`
3. Read all `parts/*/master.md` and `parts/*/schema.ts` files
4. Read all files in `tests/`
5. Generate code into `src/` that satisfies every contract
6. Run the test suite
7. Fix generated code until all tests pass

A LEAP-compatible agent MUST NOT:

- Modify any file outside `src/`
- Read `_original/` during generation (only during rewrite setup)
- Copy code from external sources (npm, GitHub, etc.) into `src/`
- Skip tests or declare them optional
- Ask clarifying questions before attempting generation (attempt first, ask only when truly blocked)

## Test authority rule

If a test contradicts a prompt, **the test wins.**

Prompts describe intent in natural language and are inherently fuzzy. Tests are executable and precise. When ambiguity arises, defer to tests.

## Part independence rule

Parts must be independently generatable. Part A's prompt must never reference Part B's implementation. Parts communicate only through:

- Shared types in `schemas/`
- Their own `schema.ts` files
- The wiring code in `src/app.*`

This enables:
- Regeneration of individual parts without touching others
- Parallel generation by multiple agents
- Clean mental model of component boundaries

## Recursion

Parts may contain their own `parts/` subdirectory for hierarchical decomposition. The same rules apply at every level.

## Self-contained src/

Generated code in `src/` must not import from outside `src/`. All contract types should be inlined or copied as needed. This ensures `src/` is a standalone, runnable artifact.

## Reproducibility, not determinism

LEAP does not require deterministic generation. Two runs of the same prompts may produce structurally different code. Correctness is defined by test results, not by code structure.

This implies:
- Snapshot tests that lock implementation are anti-patterns
- Tests must validate behavior, not implementation
- Two green generations are equally valid outputs

## Compliance checklist

A repository is LEAP-compliant if:

- [ ] `master.md` exists at the root
- [ ] `schemas/` contains at least one contract file
- [ ] `parts/` contains at least one part with `master.md` and `schema.ts`
- [ ] `tests/` contains runnable behavioral tests
- [ ] `src/` is in `.gitignore`
- [ ] Running the LEAP generation flow produces passing tests
- [ ] No application code is committed to the repo

## Version

LEAP Spec v0.1 — April 2026
