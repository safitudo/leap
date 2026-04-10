# LEAP Examples

Proof-of-concept projects built with LEAP. Each contains zero application code — only prompts, schemas, and tests.

## [semver-leap](https://github.com/safitudo/semver-leap)

Rewrite of [npm/node-semver](https://github.com/npm/node-semver), the semver parser used by npm itself.

- **5,632 / 5,632 tests passing** (upstream test suite, ported verbatim)
- 717 lines of prompts + schemas → 2,540 lines of working code
- 3.5× leverage ratio
- Slightly shorter and more idiomatic than the original in places
- Generated in one session

**Claim:** A critical JavaScript infrastructure library, rewritten in a single session, with identical behavior to the 15-year-old original.

## [todolist-leap](https://github.com/safitudo/todolist-leap)

The original LEAP proof of concept. A simple todo list web application.

- Vanilla TypeScript, no frameworks
- localStorage persistence
- Full CRUD with filtering

**Purpose:** The smallest possible LEAP project. Use this to learn the structure.

## Coming soon

Following the "daily stunt" cadence:

- **ms** — time string parser (vercel/ms)
- **chalk** — terminal color library
- **Day.js** — lightweight moment.js alternative
- **moment.js** — the classic, with honest scoping
- **lodash subsets** — array, string, object utilities
- **uuid** — cross-language proof (JS + Python + Rust from same spec)
- **ELIZA** — historical chatbot, for fun

---

Want to LEAP-ify an open source library? See [../SPEC.md](../SPEC.md) and the [leap-skill](https://github.com/safitudo/leap-skill) plugin.
