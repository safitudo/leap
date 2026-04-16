# LEAP

> **L**LM **E**ngineered **A**pplication **P**attern
>
> A development philosophy where **code is a commodity** and **guardrails are the product**.

## The pitch

Stop writing code. Write the specification.

A LEAP project contains no application code. Instead, it contains:

- **Prompts** — declarative intent
- **Schemas** — contracts between components
- **Tests** — behavioral guardrails

Any AI agent can generate the code on demand. The code is gitignored. The prompts, schemas, and tests are the source of truth.

## Proof that this works

We rewrote [npm's semver parser](https://github.com/npm/node-semver) this way:

| | [semver-leap](https://github.com/safitudo/semver-leap) | node-semver |
|---|---|---|
| Source LOC | 2,540 | 2,271 |
| Tests passing | **5,632 / 5,632** | 5,632 / 5,632 |
| Development time | **~1 session** | 15 years |
| Contributors | 1 human + AI | 106 humans |
| Human-authored lines | **717** (prompts + schemas) | 2,271 (code) |

One session. 717 lines of spec. A production-grade semver parser.

## The thesis

The internet is benchmarking AI on "how well does it write code."

That's the wrong question.

**Code is a commodity.** Well-specified libraries aren't novel algorithms — they're 15 years of "fix edge case X, add test for Y." Those edge cases are already in tests. If they're in tests, AI can eat them.

The real questions are:

1. How good are your **schemas** at defining boundaries?
2. How good are your **tests** at catching when generated code is wrong?
3. How good are your **prompts** at conveying intent?

Value shifts from **writing code** to **specifying behavior**. Tests aren't a safety net — they're the product.

## The Guardrails Hierarchy

Three artifacts, ranked by value:

1. **Tests** — most valuable. Human-authored. The definition of correctness.
2. **Schemas** — the architecture. Contracts between parts.
3. **Prompts** — least valuable. Disposable. Any prompt that passes the tests is valid.

If you lose a prompt but keep the tests and schemas, any new prompt that passes is equally correct.

## Quick start

### 1. Install the LEAP skill (Claude Code)

```
/plugin marketplace add safitudo/leap-skill
/plugin install leap-skill@leap-skill
```

### 2. Use it on a LEAP project

```bash
git clone https://github.com/safitudo/semver-leap.git
cd semver-leap
npm install

# In Claude Code:
claude "build the app, start from master.md"
```

### 3. Or start your own

Read [SPEC.md](SPEC.md) for the project structure.
Read [MANIFESTO.md](MANIFESTO.md) for the philosophy.
Read [AGENTS.md](AGENTS.md) if you use a non-Claude agent.
Read [PIXEL_PERFECT.md](PIXEL_PERFECT.md) if you're building UI.

## Examples

- **[semver-leap](https://github.com/safitudo/semver-leap)** — npm semver parser, 5,632/5,632 tests passing
- **[todolist-leap](https://github.com/safitudo/todolist-leap)** — the original proof of concept

More proofs coming daily. Follow [@safitudo](https://github.com/safitudo) for updates.

## Is this for real?

Yes. The semver-leap repo is a real working package that passes the upstream test suite verbatim. You can run `npm test` yourself.

## The bigger bet

We think most open source libraries can be generated this way. Not because AI is magic, but because:

- Most libraries aren't novel algorithms
- 15 years of bug fixes is mostly captured in test suites
- Test suites ARE the spec for "correct behavior"
- If the spec is complete, the implementation is mechanical

LEAP is betting that **the spec is the thing**. Code is what falls out.

## License

MIT

## Roadmap & contributing

- **[ROADMAP.md](./ROADMAP.md)** — what's proven, what's in flight, and the unsolved front (UI / pixel-perfect). Open invitations for library stunts, cross-model verification, and UI experiments.
- **[CONTRIBUTING.md](./CONTRIBUTING.md)** — how to propose SPEC/MANIFESTO edits, submit examples, or port another library as a stunt.
- **[Discussions](https://github.com/safitudo/leap/discussions)** — open-ended questions, counter-arguments, "what if LEAP worked like X?"
- **Issues** — actionable proposals and bugs in the methodology.

## Home

[leapagentic.io](https://leapagentic.io) — the home of LEAP.
