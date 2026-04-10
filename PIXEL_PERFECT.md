# LEAP for Pixel-Perfect UI

> How LEAP handles frontend projects where the rendered output must match a design exactly.

## The central idea

Turn "looks right" into an automatable test.

In traditional frontend workflows:

1. Designer produces a reference (Figma)
2. Developer looks at it, writes code
3. Developer looks at the result, tweaks
4. PM looks at both, says "move the button 2px"
5. Repeat until everyone nods

The feedback loop is human-in-the-loop and slow. AI coding assistants can't participate meaningfully because they can't see.

LEAP inverts this:

1. Designer produces a reference (Figma export or PNG)
2. Reference becomes a test fixture in `design/references/`
3. A visual test compares rendered output to the fixture with a pixel diff
4. AI iterates on the code until the diff passes
5. A human reviews once, at the end (if at all)

**The human feedback loop becomes a machine feedback loop.** AI gets visual feedback via pixel diffs and can iterate without guessing.

## Why this works now

Four things have quietly become production-ready:

1. **Playwright / Puppeteer pixel comparison** — robust, cross-platform, configurable thresholds
2. **Headless Chromium** — deterministic rendering when pinned
3. **Design tokens** — machine-readable design systems (Style Dictionary, Tokens Studio)
4. **Figma API exports** — reference images as a build artifact

None of these are new. What's new is treating them as the **primary specification** instead of supporting tooling.

## Project structure

```
ui-project/
├── master.md                      # project description + tech constraints
├── design/                        # THE SPECIFICATION
│   ├── tokens.json                # design system (machine-readable)
│   ├── references/                # one PNG per component per state
│   │   ├── button-primary-default.png
│   │   ├── button-primary-hover.png
│   │   └── ...
│   ├── breakpoints.json           # responsive breakpoint config
│   └── figma-export.json          # optional: source of truth link
├── parts/
│   └── button/
│       ├── master.md              # component intent
│       ├── schema.ts              # props/API contract
│       └── states.json            # state → reference image mapping
├── tests/
│   ├── visual/                    # pixel diff tests
│   ├── interaction/               # behavior tests
│   └── a11y/                      # accessibility tests
├── playwright.config.ts           # pinned browser version
└── src/                           # GITIGNORED
```

## Reference image formats

### When to use PNG

**Default choice.** Pixel-perfect comparison. Captures:
- Final rendered appearance
- Font rendering (subpixel AA, kerning)
- Gradients, shadows, blur
- Real browser output

Trade-off: locked to a specific rendered size. A 200×60 button reference can only verify a 200×60 button.

### When to use SVG

For vector-based comparisons, layout structure, icons. More flexible with size but less pixel-accurate.

### When to use multiple PNGs

For responsive components: `button@mobile.png`, `button@tablet.png`, `button@desktop.png`. Each tested at the corresponding viewport.

### File naming convention

```
{component}-{variant}-{state}[@{breakpoint}].png
```

Examples:
- `button-primary-default.png`
- `button-primary-hover.png`
- `button-ghost-focus.png`
- `card-default@mobile.png`
- `card-default@desktop.png`

This naming is parseable by tests and prompts.

## Diff thresholds

Never use exact pixel matching. Font rendering varies slightly even on pinned browsers due to font cache, DPI, and OS-level anti-aliasing.

### Recommended thresholds

| Component type | `maxDiffPixelRatio` | Rationale |
|---|---|---|
| Buttons, inputs, chips | 0.005 (0.5%) | Small, predictable |
| Cards, panels | 0.01 (1%) | Some variance in inner content |
| Full page layouts | 0.02 (2%) | Many small sources of variance |
| Text-heavy components | 0.03 (3%) | Font rendering drifts |

### Threshold philosophy

A threshold below 0.5% means "any meaningful change breaks this test." Above 3% means "small regressions slip through." Tune per component based on its visual complexity.

## Cross-browser rendering strategy

### Pin the browser

```ts
// playwright.config.ts
export default defineConfig({
  use: {
    // Pin to a specific Chromium version
    channel: "chromium",
  },
  // Match CI and local exactly
});
```

### Run visual tests in containers only

Reference images should only be generated on the container image that CI uses. Do not commit references generated on a developer laptop — OS font rendering will drift.

### Bootstrapping references

The first time you create a component, generate the reference from a trusted source:
1. Export from Figma at exact pixel dimensions, OR
2. Run the test in "record mode" in a pinned container, OR
3. Hand-craft the reference from design tokens

Then commit the reference. All future runs compare against it.

## Interactive states

Most components have multiple visual states. Each needs a reference.

### Common states per component

- `default` — resting state
- `hover` — pointer over
- `focus` — keyboard focus visible
- `active` — being pressed
- `disabled` — non-interactive
- `loading` — async state (if applicable)
- `error` — validation failure (if applicable)

### Triggering states in tests

```ts
test("button hover matches reference", async ({ page }) => {
  await page.goto("/button?variant=primary");
  await page.hover("button");
  await expect(page.locator("button")).toHaveScreenshot("button-primary-hover.png");
});
```

The test does what a user would do. The AI's job is to make the rendered output match.

## Responsive design

### Viewport references

Each responsive component gets a reference per breakpoint:

```
design/references/
├── card-default@mobile.png      # 375×X
├── card-default@tablet.png      # 768×X
├── card-default@desktop.png     # 1280×X
```

### Tests run at each viewport

```ts
const viewports = [
  { name: "mobile", width: 375 },
  { name: "tablet", width: 768 },
  { name: "desktop", width: 1280 },
];

for (const vp of viewports) {
  test(`card at ${vp.name}`, async ({ page }) => {
    await page.setViewportSize({ width: vp.width, height: 800 });
    await page.goto("/card");
    await expect(page).toHaveScreenshot(`card-default@${vp.name}.png`);
  });
}
```

## Motion and animation

**Open problem.** Current approaches:

1. **Freeze animations** — disable CSS transitions in tests, capture static states
2. **Timed screenshots** — capture at specific animation progress (e.g., 0%, 50%, 100%)
3. **Video references** — record animation, diff frame-by-frame (expensive, flaky)

Recommended for now: freeze animations and test the start/end states as separate references. Full motion testing is a v0.3+ concern.

## The radical promise

If the `design/` directory is complete and the tests are thorough:

- **AI never needs human review for correctness.** The tests are the review.
- **Design system changes become regenerations.** Update tokens, regenerate, done.
- **Framework migrations are trivial.** Same references, same tests, different generated output.
- **Multi-framework targeting is free.** React, Vue, Svelte — one spec, three implementations, identical pixels.

That last point is genuinely new. No frontend methodology has ever allowed that.

## The honest caveats

1. **Reference creation is manual work.** Someone has to export from Figma or craft the PNGs. LEAP doesn't eliminate this — it formalizes it.

2. **Flaky visual tests are real.** Subpixel drift, font variations, container timing. You will spend time tuning thresholds.

3. **Motion is unsolved.** If your UI is animation-heavy, LEAP can verify start/end states but not transitions.

4. **"Vibes" designs don't fit.** If the design is "it should feel clean and modern," there's no test. LEAP forces precision in design specs, which is both a feature and a limitation.

5. **Accessibility beyond automated tests still needs humans.** Axe catches violations, but screen reader UX needs a person.

## Getting started

Minimum viable LEAP UI project:

1. Create one component (a Button)
2. Export 4 reference PNGs: default, hover, focus, disabled
3. Create `design/tokens.json` with colors and spacing
4. Write 4 visual regression tests
5. Write 2 interaction tests (click fires handler, disabled doesn't)
6. Write 1 a11y test (WCAG contrast)
7. Write `parts/button/master.md` describing the intent
8. Point an AI agent at it
9. Verify tests pass

If that works, scale up. If it doesn't, the framework needs tuning before bigger projects.

---

**Status:** Draft methodology. Needs validation via an actual LEAP UI project. See [examples](examples/) for available proofs of concept.
