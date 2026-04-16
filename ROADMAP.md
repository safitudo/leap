# LEAP Roadmap

> LEAP is experimental by design. This roadmap tracks what we've proven, what we're working on now, and what's genuinely unsolved. It updates as we learn.

**Last updated:** 2026-04-16

---

## Current state — what's proven

- **Methodology core** — specs + schemas + tests as source, code as generated artifact. Documented in [SPEC.md](./SPEC.md) and [MANIFESTO.md](./MANIFESTO.md).
- **Library-scale code** — [semver-leap](https://github.com/safitudo/semver-leap) rewrote npm's `node-semver` end-to-end. 717 lines of specs → 2,540 lines of generated code. 5,632 / 5,632 upstream tests passing. Regenerated twice from scratch, both green.
- **Agent integration** — [leap-skill](https://github.com/safitudo/leap-skill) encodes the workflow as a Claude Code plugin.
- **Audit tooling** — [leap-audit](https://github.com/safitudo/leap-audit) provides a 12-dimension scorecard for assessing AI-delivery readiness on any codebase.
- **Operational stack** — [leap-ops-stack](https://github.com/safitudo/leap-ops-stack) documents the surrounding ops layer (vault, MCP, transcription).

---

## Near-term (Q2 2026) — expanding the proof

The single data point of `semver-leap` is not enough. We want a portfolio of stunts across domains to map where LEAP holds and where it breaks.

### Library stunts — port more real code

Each of these is an invitation. If you want to pick one up, open an issue titled `[stunt] <name>` and go — we'll link it here on completion.

| Target | Why it matters | Status |
|---|---|---|
| `ms` (vercel/ms) | Smallest interesting lib. Fast feedback on LEAP for micro-utilities. | Open |
| `chalk` | Terminal color library. Tests are visual-ish; will stress the "behavior-describable-as-tests" claim. | Open |
| `Day.js` / `date-fns` subset | Date math with real-world edge cases (timezones, leap years). Bigger than semver. | Open |
| `uuid` cross-language | Same specs → generated impl in TS, Go, Rust. Validates agent-agnostic claim. | Open |
| `lodash` subset (10–20 fns) | Functional utilities. Tests map cleanly. | Open |
| SQLite subset (core parsing) | Stretch goal. Real systems code. Major stress test. | Later |

**What "success" looks like:** for each port, document lines-of-spec → lines-of-code, test-pass ratio, failure modes encountered, and honest commentary on what was hard.

### Cross-model verification

The same specs should generate equivalent code across Claude, GPT-5.x, and Gemini 3. We've only battle-tested Claude Code. Open call: regenerate `semver-leap` with another model, report what broke.

---

## The unsolved front — UI / pixel-perfect

**This is the most important open problem in LEAP.** The thesis — tests describe behavior, code is disposable — gets wobbly when the product is visual.

- Tests can describe **interaction behavior** (clicks, keyboard, focus order, form validation, error states). That part works.
- Tests cannot describe **aesthetic intent** (spacing, rhythm, typography hierarchy, brand-specific polish). Visual regression testing is a patch, not a framework.
- Design systems reduce the surface but don't solve it — the design system itself is an ambiguous spec.

We've started thinking about this in [PIXEL_PERFECT.md](./PIXEL_PERFECT.md). Current sketch:

- Schemas for component contracts (props, slots, states, accessibility)
- Behavior tests for interaction logic
- Visual regression + design tokens for aesthetic fidelity
- Prompts that reference design-system primitives instead of re-describing them

**What we don't know yet:**
- How do you prompt for "the right feel" of a landing hero when the spec can't name every coordinate?
- At what scale does the cost of visual regression maintenance cross the break-even line vs. hand-writing CSS?
- Do agentic tools with DOM/screenshot feedback loops (Claude Code browser use, Cursor vision) close the gap, or create new failure modes?

**Explicit invitation:** if you've solved any slice of this, we want PRs to `PIXEL_PERFECT.md` and issues with concrete experiments. This is where LEAP's next breakthrough lives.

---

## Longer-term (H2 2026+)

- **Integration-heavy code** — when the "spec" is "match an external API exactly," the spec is functionally the API doc. Need a pattern that acknowledges this and doesn't duplicate the API contract by hand.
- **Scalable UIs at scale** — moving from single-component pixel-perfect to multi-screen flows with real data, A/B surfaces, and feature flags. Where does the LEAP artifact graph live?
- **Multi-repo coordination** — a LEAP project today is one repo. What happens with microservices, monorepos, cross-team contracts?
- **Tooling that makes LEAP operations cheap** — regeneration diff review, spec-drift detection, schema-vs-test coverage mapping.

---

## How to influence this roadmap

- **Discussions** on this repo for open-ended questions ("what if LEAP worked like X?").
- **Issues** for concrete proposals or bugs in the methodology.
- **PRs** for direct edits to SPEC.md, MANIFESTO.md, examples, or new stunts in their own repos linked here.
- **Stunt ports** — pick anything from the library table above, ship the repo, PR a link.

LEAP is only as strong as the communities beating on it. The roadmap is a living document — expect it to move as we learn what we got wrong.
