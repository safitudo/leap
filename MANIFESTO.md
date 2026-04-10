# The LEAP Manifesto

> Code is a commodity. Guardrails are the product.

## We believe

### 1. Code is not the thing.

For decades, we've treated source code as the primary artifact of software development. We version it, review it, protect it, maintain it, refactor it, rewrite it. We argue about style guides, bikeshed over naming, and build entire careers around code craftsmanship.

But code is downstream of intent. It is the manifestation, not the essence. The essence is:

- **What should the software do?** (intent)
- **How do the parts fit together?** (architecture)
- **How do we know it's correct?** (specification)

Code is the answer to those questions. But the questions are what matter.

### 2. AI changed the economics of code.

Writing syntactically correct, idiomatic, reasonably performant code is no longer the bottleneck. Any competent LLM can produce working code from a clear specification.

What remains hard:

- **Knowing what to build.**
- **Catching subtle mistakes.**
- **Deciding when it's done.**

These are human problems. Code generation is not.

When the expensive part of a process becomes cheap, value migrates upstream. The upstream of code is: **prompts, schemas, and tests.**

### 3. Tests are the real source of truth.

Every mature open source library tells the same story: 95% of its code history is bug fixes, each one capturing an edge case someone tripped over. That accumulated wisdom lives in the test suite.

node-semver has 2,271 lines of code and 5,632 test assertions. Which is the product?

We argue: **the tests are the product.** The code is an implementation detail that any sufficiently capable generator can produce.

### 4. Guardrails are the new craftsmanship.

If code is cheap and tests are the spec, then the craft of software engineering becomes:

- Writing **tests** that capture correct behavior without over-specifying implementation
- Designing **schemas** that define crisp boundaries between components
- Authoring **prompts** that communicate intent clearly

This is engineering at a higher level of abstraction. Less typing, more thinking. Less code review, more spec review.

### 5. Generation should be reproducible, not deterministic.

Two runs of the same prompt may produce structurally different code. That is a feature, not a bug.

If the tests pass, the code is correct — regardless of whether the AI chose a for-loop or a reduce. The spec defines correctness; the implementation is free to vary.

This inversion unlocks things traditional codebases can't do:
- Regenerate everything with a better model and get better code for free
- Target multiple languages from the same spec
- Detect spec ambiguity when different generations disagree

### 6. The guardrails hierarchy.

Three artifacts, ranked by value:

1. **Tests** (most valuable)
   Human-authored. Behavioral. The definition of "correct."
   Lost tests = lost ground truth.

2. **Schemas** (architecture)
   The contracts between components.
   Lost schemas = lost structure.

3. **Prompts** (least valuable)
   The intent. Disposable.
   Lost prompts = minor setback. Any new prompt that passes the tests is equally valid.

Maintain in reverse order of value. Spend the most time on tests.

### 7. Code lives in src/. src/ is gitignored.

The repo contains only what humans author. Generated code is an artifact, like a compiled binary. We don't commit binaries; we don't commit generated code.

This forces discipline:
- You can't sneak in manual edits that drift from the spec
- The spec must actually be sufficient, not "sufficient plus the code you wrote by hand"
- Regeneration is always the answer, never a fallback

### 8. LEAP is a pattern, not a product.

LEAP is the pattern. Frameworks, tools, and agents can implement it. We encourage that. This document and the accompanying spec are the lingua franca — use them however you like.

If it reads `master.md`, generates code from prompts, and runs tests — it's LEAP.

## We reject

- **The primacy of code.** Code is downstream.
- **The veneration of craft.** Craft is what you apply, not what you produce.
- **AI benchmarks that measure code quality in isolation.** That's printer-paper quality. Measure spec quality instead.
- **The idea that rewrites are expensive.** Rewrites are a button press when the spec is separate from the code.

## We embrace

- **Disposable code.** Every session starts clean.
- **Specs as first-class artifacts.** Treat your tests like you treat your schema migrations.
- **AI agents as compilers.** Feed them a spec, get code.
- **Agent portability.** LEAP doesn't care which model you use.

## The future we're building

Every open source library, rewritten from its own test suite. Smaller, cleaner, faster, maintained by specifications rather than accumulated patches.

A world where starting a new project means writing tests first, schemas second, and generating the rest.

A world where code is genuinely disposable, because the thing that matters — the specification — is what you commit to memory, to git, and to history.

Welcome to LEAP.
