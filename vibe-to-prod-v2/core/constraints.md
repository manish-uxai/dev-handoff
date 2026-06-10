# Core Constraints

> Load this file on every invocation. These three rules are non-negotiable regardless of stack, mode, or scope.

---

## Constraint 1: Preserve the design surface

The **design surface** — layout, spacing, visual hierarchy, interaction behavior, animation feel — is sacred. Never alter it.

The **engineering substrate** — how animations are implemented, how state flows, how components are structured internally — can and should be improved, as long as the perceived result is indistinguishable or better.

**Allowed:**
- Refactoring a janky Framer Motion spring config to one that produces the same feel without frame drops
- Fixing a CSS specificity bug causing hover flicker
- Splitting a 600-line component into smaller ones that render identically

**Not allowed:**
- Simplifying an animation because it "looks complex"
- Changing spacing or layout while refactoring state
- Removing a transition because it's not in a standard library

**When an interaction looks buggy rather than intentional — flag it, don't silently fix or preserve it.**

Code-detectable interaction smells:
- Framer Motion `stiffness` > 500 or `damping` < 5 — likely jank
- CSS transitions with `0ms` or `0s` duration — likely missing
- Hover/focus handlers toggling state without debounce — likely flicker
- `setTimeout`/`setInterval` driving visual transitions — likely a hack

---

## Constraint 2: No direct data in components

Components must never import mock data arrays directly or call `fetch()` internally. All data must flow through async functions in `api/index.ts` or `api/index.js`.

This applies to every component without exception:
- Maps fetching GeoJSON directly — violation
- Charts loading CSV internally — violation
- Tables importing a `MOCK_DATA` array — violation

This rule is all-or-nothing. One violation = dimension 4 fails entirely.

When a developer connects real APIs, they change only the function body inside `api/index.ts`. Every component updates automatically.

---

## Constraint 3: The output is production-ready code

The deliverable is clean, production-survivable code. Not a documentation package. Not a list of what was changed.

- No separate contract files to generate
- No "replaced components" lists
- No migration guides as output
- The `// @backend` annotations in `api/index.ts` are the only integration reference a developer needs — they live in the code

---

## Unsupported stacks

**Plain HTML/CSS/JS (no `package.json`):**

> "Your project is built with plain HTML and CSS. This skill is designed for React-based projects — the component structure, data flow patterns, and production-readiness checks are all React-specific.
>
> If we convert your prototype to React first, everything gets structured and ready — components, data flow, API connections — and a developer can skip straight to integration. That could save them 4-6 weeks.
>
> Want me to help convert your HTML prototype to React?"

**Vue, Svelte, Angular, or other frameworks:**

> "Your project uses [detected framework]. This skill is built around React and Next.js, so running it here would give inaccurate results.
>
> Two options: I can help migrate to React so the full pipeline works, or I can give general handoff advice without the structured audit. Which works better?"
