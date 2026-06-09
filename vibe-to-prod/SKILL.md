---
name: vibe-to-prod
description: Transforms high-fidelity designer-built React or Next.js prototypes into production-ready, frontend-only codebases. Detects the project stack first and either proceeds, adapts, or guides the user to the right path. Use this skill when a designer wants to hand off their coded prototype to a frontend developer, when someone wants to make their vibe-coded UI "dev-ready", when asked to audit or refactor a prototype for backend integration, or when the goal is to eliminate UI re-development by treating the prototype as the final production frontend. Triggers on phrases like "handoff to dev", "make this production ready", "clean up my prototype", "prep for backend integration", "frontend handoff", "vibe to prod", or any mention of sharing a codebase with a frontend developer.
license: MIT
compatibility: Works with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, and other agentskills.io-compatible agents. Supports React and Next.js projects. Other stacks trigger guided redirection.
metadata:
  author: vibe-to-prod
  version: "3.3.0"
  framework: 18-dimension-handoff
---

# Philosophy: Figma-less Direct-to-Code Handoff

Traditional handoff is broken. A designer spends weeks — sometimes months — building a polished, high-fidelity prototype in code. Then they hand over a Figma file or screenshots. The frontend developer spends another 1–2 months rebuilding the exact same UI from scratch. Fidelity degrades. Micro-interactions get lost. Time is wasted twice.

**We break this cycle entirely.**

The designer-builder has already developed the final, high-fidelity production UI. Your job is to treat this prototype codebase as the **actual production frontend** and harden the architecture underneath it — so a frontend engineer can open this project, plug in real APIs, and ship. Zero visual rewrites. Zero animation loss. Zero layout shifts. **1–2 months of UI development saved.**

You are not "cleaning up a throwaway demo." You are making the invisible infrastructure production-ready while leaving everything the user can see and feel completely intact.

---

# Step 0: Stack Detection

**Before anything else, identify what the user has built.** Do not assume React. Check `package.json`, file extensions, and folder structure. Then follow the response path below.

---

## React (with TypeScript) — `.tsx` / `.ts`
Proceed directly with all 18 dimensions. This is the ideal path.

---

## React (with JavaScript) — `.jsx` / `.js`
Fully supported. Proceed directly with all 18 dimensions — the skill automatically applies the JavaScript-appropriate approach for each one. No warnings, no recommendations unless the user asks.

---

## Next.js — `next.config.*` present or `next` in `package.json`
Proceed with all 18 dimensions, but apply the Next.js overrides in the section below. Say something like:
> "Your project is built with Next.js — great choice. This skill works well with Next.js. A few things work differently here so I'll adapt as we go."

---

## Plain HTML / CSS / JS — no `package.json`, or only `.html` / `.css` files
Do not proceed with the audit. Explain clearly and offer a path forward:

> "Your project is built with plain HTML and CSS, which is a great way to prototype quickly. This skill is designed for React-based projects — here's why that matters for your handoff:
>
> Right now, a developer would need to manually convert your HTML into components, set up routing, connect APIs, and build out all the interactions from scratch. That's the exact problem this skill is designed to avoid.
>
> If we convert your prototype to React first, your developer gets everything structured and ready — components, data flow, API connections — and can skip straight to integration. That's roughly 4-6 weeks of work they won't have to do.
>
> Would you like me to help convert your HTML prototype to React so we can make it fully handoff-ready?"

---

## Vue.js, Svelte, Angular, or other frameworks
Do not proceed with the audit. Explain clearly:

> "Your project is built with [detected framework]. This skill is currently built around React and Next.js — the handoff pipeline, component patterns, and API contract structure are all React-specific, so running it on a [framework] project would give you inaccurate results.
>
> You have two options:
> 1. I can help you migrate this to React so the full handoff pipeline works
> 2. We can do a manual review of your project structure and I can give you general handoff advice, though it won't be as precise
>
> Which would work better for you?"

---

## Language and tone rule (applies to all stack responses)

**Always communicate in plain, friendly language.** This skill is used by designers and PMs, not just developers. Avoid technical jargon unless the user is clearly technical. Translate everything:

| Instead of this | Say this |
| :--- | :--- |
| "dimension 14 blocker" | "there's one thing worth fixing before handoff" |
| "JSDoc typings" | "an older way of describing your data" |
| "reducer isolation" | "a cleaner way to manage all your app's moving parts" |
| "TypeScript interface" | "a description of what your data looks like" |
| "god-component" | "one file that's doing too many things at once" |
| "API contract stubs" | "placeholder connections where real data will flow in" |
| "hardcoded hex values" | "colors written directly into the code instead of using your design system" |

Read the user's messages for cues. If they use terms like "component", "props", or "state" correctly, you can be more technical. If they say "my app" or "my design", stay plain.

**Mandatory language enforcement:** Before writing ANY audit finding, violation, or recommendation to a non-technical user, rewrite it using the translation table above. If the user has not used developer jargon in their messages, every finding in the Issue and Recommended Fix sections must be in plain language. Reserve file paths, line numbers, and grep output for the Evidence column only. Never output raw dimension numbers or framework jargon in prose unless the user is clearly technical.

---

# Core Constraints

## 1. Preserve Design Intent, Allow Implementation Improvements

The **design surface** — layout, spacing, visual hierarchy, interaction behavior, animation feel — is sacred. Do not alter it.

The **engineering substrate** — how animations are implemented, how state flows, how components are structured internally — can and should be improved, as long as the perceived result is indistinguishable or better.

Concretely:
- A Framer Motion spring with a config causing frame drops → refactor the config, preserve the feel
- An animation triggering unnecessary re-renders → fix the implementation, keep the motion
- A hover state that flickers due to a CSS specificity bug → fix it, that's not intentional design
- A layout that renders identically after a component split → safe to refactor

**When a vibe-coded interaction appears buggy rather than intentional, surface it to the developer as a flag rather than silently preserving or silently fixing it.**

**Code-detectable interaction smells (no browser required):**
- Framer Motion springs with `stiffness` > 500 or `damping` < 5 (likely jank)
- CSS transitions with `0ms` or `0s` durations (likely missing)
- Hover/focus handlers toggling state without debounce (likely flicker)
- Animation components depending on frequently re-rendering parent state (likely frame drops)
- `setTimeout`/`setInterval` driving visual transitions instead of CSS or animation libraries

## 2. Component Replacement Strategy: Prefer Swappable Primitives

Vibe-coded prototypes often hand-roll UI primitives — dropdowns, modals, tooltips, tabs, date pickers — because the AI generated them inline. These look correct in demo conditions but lack keyboard navigation, scroll-locking, portal rendering, focus trapping, and edge-case resilience.

**Distinguish two types of custom components:**

| Type | Definition | Action |
| :--- | :--- | :--- |
| **Genuine custom** | A novel interaction or visualization specific to this product | Preserve and harden |
| **Reinvented primitive** | A hand-rolled dropdown, modal, tooltip, tabs, etc. | Replace with headless library equivalent |

For reinvented primitives, replace with **shadcn/ui**, **Radix UI**, or **Headless UI** components styled to match the designer's exact visual output. Do not use raw custom implementations.

**Why shadcn specifically:** When a frontend dev integrates their own design system (e.g., Storybook component library), replacing a shadcn component is a clean prop-to-prop swap. Replacing a custom vibe-coded component requires reverse-engineering intent first. shadcn gives the dev a predictable, well-documented swap surface.

## 3. Enforce Component Reusability

Vibe-coding produces duplicated components — the same button, card, badge, or input rebuilt slightly differently across multiple files. Before a dev can replace anything with their own design system, they need one component to replace, not twelve variants.

**Scan the entire codebase for repeated UI patterns and consolidate them into shared primitives in `components/ui/`.** A developer should be able to swap one `<Button />` import and have it propagate everywhere.

## 4. API Stubs Are Mandatory — No Direct Mock Data in Components

Components must never import mock data arrays directly. All data must flow through typed async functions in `api.ts`, even if those functions return mock data internally.

**This is a hard rule, not guidance.** It applies to every component without exception — including maps fetching GeoJSON, charts loading CSV/JSON, tables pulling datasets, and any third-party visual integration that makes its own network requests. If a component calls `fetch()` or imports data directly, it violates this rule regardless of what it renders.

When the developer connects real APIs, they change only the function body inside `api.ts`. Every component that consumes that data updates automatically — no component-level changes required.

---

# The 18-Dimension Handoff Framework

## Structure & Separation

### 1. Component Architecture & DOM Hierarchy Guard

- Isolate stateful logic from presentational components. Enforce one domain context per provider.
- Break massive "god-components" (400+ lines) into focused, presentational child modules.
- **DOM Hierarchy Guard:** Preserve the exact DOM layout hierarchy when splitting components. Do not introduce redundant wrapper elements or alter CSS display properties (`flex`, `grid`) of parent-child relationships — this instantly breaks style containment and positioning.

### 1b. Component Reusability Pass (separate dimension — never merge with 1)

This is a **mandatory separate audit step**. Do not fold reusability findings into dimension 1. Report dimension 1 and 1b as distinct rows in every audit.

**Detection methods (run all that apply):**

1. **Duplicate export names** — same component name exported from multiple files:
   ```bash
   rg "^export (default )?(function|const) \w+" src/components src/pages --glob '*.{tsx,jsx}' -o | sort | uniq -d
   ```
2. **Similar file names** — `Button.tsx`, `CustomButton.tsx`, `PrimaryButton.tsx` in different folders:
   ```bash
   find src -iname '*button*' -o -iname '*card*' -o -iname '*badge*' | sort
   ```
3. **Repeated JSX prop shapes** — same prop interface rebuilt (e.g., `variant`, `size`, `onClick`) across files without a shared import:
   ```bash
   rg "variant.*size|size.*variant" src/components --glob '*.{tsx,jsx}' -l
   ```
4. **className-only duplication** — long identical class strings (weak signal; use with methods 1–3):
   ```bash
   rg -o 'className="[^"]{60,}"' src --glob '*.{tsx,jsx}' | sort | uniq -d
   ```

In audit mode: list every duplicated pattern with **all file paths**, then state what shared primitive in `components/ui/` would replace them. In refactor mode: consolidate before marking 1b passed.

### 2. Clean Data Extraction

- Move all `MOCK_` arrays, seed data, and hardcoded lists to `/src/data/`. No inline arrays in UI components.
- Preserve the exact data shape to prevent rendering errors or layout discrepancies.

### 3. Canonical Domain Types (`domain.ts` / `domain.js`)

The goal is a single file that describes what all your data looks like — so a developer knows exactly what to expect from every API response without reading through the whole codebase.

**If React + TypeScript:**
Create `domain.ts` with strict TypeScript interfaces for every entity the UI renders. Developers use these to validate backend API contracts at compile time.

```ts
export interface Patient {
  id: string;
  name: string;
  status: 'active' | 'inactive';
}
```

**If React + JavaScript:**
Create `domain.js` with JSDoc type definitions for every entity. Not as strict as TypeScript, but gives the developer a clear map of the data structure and enables IDE autocomplete.

```js
/**
 * @typedef {Object} Patient
 * @property {string} id
 * @property {string} name
 * @property {'active' | 'inactive'} status
 */
```

In both cases: one file, one source of truth, no duplicated or conflicting type definitions across the codebase.

**If React + JavaScript — JSDoc quality (not just presence):**
Do not pass dimension 3 because `domain.js` exists. Evaluate quality:

- Every entity the UI renders has a `@typedef` with all properties used in components
- Property types match actual runtime usage (no `@property {string} id` when code uses `id` as number)
- No duplicate `@typedef` names across files
- `@param` and `@returns` on every function in `api.js` matching stub signatures
- Flag incomplete JSDoc as a dimension 3 violation with specific missing or wrong properties cited

### 4. API Contract Stubs (`api.ts` / `api.js`)

- **Hard rule:** No component imports mock data directly. All data flows through async functions in `api.ts` or `api.js`.
- Implement async functions simulating latency (`await delay(300)`) for all data dependencies.
- **Strict Envelope Rule:** Mock responses must use realistic response envelopes (e.g., `{ data, meta: { total, page } }`) rather than flat arrays.
- **Third-Party Integrations:** Maps, charts, heatmaps — never fetch internally. Consume data via API stubs passed through state/context.
- Every stub must carry a `// @backend` annotation.

**If React + TypeScript:**
```ts
// @backend POST /api/patients
// Auth: Bearer token (JWT)
// Payload: { name: string; status: 'active' | 'inactive' }
// Response: { data: Patient; meta: { createdAt: string } }
export async function createPatient(payload: CreatePatientPayload): Promise<ApiResponse<Patient>> {
  await delay(300);
  return { data: MOCK_PATIENTS[0], meta: { createdAt: new Date().toISOString() } };
}
```

**If React + JavaScript:**
```js
// @backend POST /api/patients
// Auth: Bearer token (JWT)
// Payload: { name: string, status: 'active' | 'inactive' }
// Response: { data: Patient, meta: { createdAt: string } }
export async function createPatient(payload) {
  await delay(300);
  return { data: MOCK_PATIENTS[0], meta: { createdAt: new Date().toISOString() } };
}
```

The structure, annotations, and envelope shape are identical. The only difference is the absence of type annotations in the JS version.

## Backend Integration & State

### 5. State Management & Reducer Isolation

- Avoid chains of 5+ independent `useState` hooks causing cascading re-renders.
- Consolidate state mutations into a reducer-based state slice with explicit action schemas, history metadata, and optional rollback hooks. Allows developers to hook up Redux or Zustand instantly.

### 6. Read-Only / Role-Based Access Control

- Guarded setters must block data mutation. Use the HTML `inert` attribute to block UI interaction for read-only roles.
- Apply `inert={isReadOnly}` to interactive form containers to freeze modifications without altering styles.
- **If TypeScript:** define roles as a strict union type in `domain.ts` (e.g., `type Role = 'admin' | 'viewer'`).
- **If JavaScript:** define roles as a plain constants object in `domain.js` (e.g., `export const ROLES = { ADMIN: 'admin', VIEWER: 'viewer' }`). Same behaviour, no compile-time enforcement.

### 7. Backend Integration Markers & Contract Sync

- Every API stub annotated with `// @backend` as shown in dimension 4.
- Ensure annotations match the data shapes defined in `domain.ts` or `domain.js` — what the stub returns must match what the file describes.
- After refactoring, generate `BACKEND_CONTRACT.md` using the schema in [references/backend-contract-template.md](references/backend-contract-template.md).

## UI Quality & Performance

### 8. Component Library Compliance

- **Actively scan for reinvented UI primitives** (dropdowns, modals, tooltips, tabs, popovers, date pickers, checkboxes, radios, select menus). Do not passively check whether existing ones are compliant — search for components that should be replaced but haven't been. Run the grep patterns from audit-checklist.md. A god-component almost certainly contains inline primitives.
- Replace reinvented primitives with **shadcn/ui** or **Radix UI** equivalents, preserving the designer's exact visual styling.
- Genuine custom components (novel interactions, product-specific visualizations) are preserved and hardened — not replaced.
- Flag any replaced component in `BACKEND_CONTRACT.md` so the developer knows which primitives have clean swap surfaces.
- **Do not auto-pass this dimension.** If no scan was performed, mark it as unevaluated, not passing.

**Mandatory audit output — two lists (non-negotiable):**

Every audit and refactor report must include both lists below. Missing either list = dimension 8 incomplete.

```markdown
### Dimension 8 — Reinvented Primitives (replace with shadcn/Radix)
| Component | File:Line | Suggested Replacement |
|-----------|-----------|----------------------|
| CustomDropdown | `src/components/Header.tsx:42` | shadcn `<Select />` |

### Dimension 8 — Genuine Custom (preserve and harden)
| Component | File:Line | Why Custom |
|-----------|-----------|------------|
| TimelineDragReorder | `src/components/Timeline.tsx:1` | Product-specific drag interaction |
```

If a list is empty, write explicitly: `None found after active scan.` Do not omit the section.

### 9. CSS Token & Design System Compliance

- Map **colors, spacing, sizing, and typography** to semantic CSS variables (e.g., `--color-status-danger`, `--spacing-container`, `--font-size-body`, `--size-icon-sm`). No hardcoded design values remaining after refactor.
- **Color violations:** hardcoded hex, `rgb()`, `hsl()`, Tailwind arbitrary colors (`bg-[#f3f4f6]`)
- **Spacing violations:** raw pixel padding/margin/gap in inline styles (`padding: 17px`), or Tailwind arbitrary spacing (`p-[17px]`, `m-[13px]`, `gap-[22px]`)
- **Sizing violations:** raw pixel width/height in inline styles or arbitrary Tailwind (`w-[243px]`, `h-[312px]`)
- **Typography violations:** hardcoded `fontSize`, `lineHeight`, `letterSpacing`, `fontWeight` in inline styles or arbitrary Tailwind (`text-[13px]`, `leading-[1.37]`)
- **Defensive Tailwind Snapping:** Scan for Tailwind arbitrary bracket values. Snap to nearest standard Tailwind scale only if the visual shift is less than `1px`. Otherwise extract to a semantic CSS variable.

### 10. Destructive Action Safety

- Gate all deletions, resets, or irreversible state changes behind a reusable `ConfirmDialog` or "Toast with Undo" pattern. No immediate destructive actions.

### 11. Routing & Navigation (`react-router`)

- Replace conditional page rendering (`if (page === 'home')`) with declarative `react-router-dom` paths.
- Implement lazy loading (`React.lazy`) for heavy route-level containers.
- Set up Route Guards (`<ProtectedRoute />`) for authenticated/role-based views.

### 12. Component Performance (Memoization)

- `useMemo` for expensive client-side filtering, sorting, or calculations.
- `useCallback` for event handlers passed to child components.
- `React.memo` on heavy list items, cards, or custom data grid rows.

## Robustness & Typing

### 13. Accessibility (a11y) Baseline

- Eliminate "div soup." Enforce semantic HTML5 landmarks (`<nav>`, `<main>`, `<header>`, `<button>`, `<aside>`).
- Interactive elements must have readable `aria-label` or `aria-labelledby` attributes.
- Keyboard focus navigation: proper `tabIndex`, visible focus states, focus trapping in modal dialogs.
- `aria-live` regions for dynamic real-time updates.
- **Note:** Replacing custom primitives with shadcn/Radix automatically resolves most a11y gaps for those components.

### 14. Linting, Formatting & Code Quality

**If React + TypeScript:**
- Eradicate `any` types. Convert legacy JSDoc typings into strict TypeScript interfaces.
- Organize imports: external libraries → internal components/hooks → styles/assets.
- Strict ESLint/Prettier compliance: no unused variables, exhaustive `useEffect` dependencies.

**If React + JavaScript:**
- Add or enforce PropTypes on all components so developers understand what each component expects.
- Ensure JSDoc comments exist on all functions in `api.js` and `domain.js`.
- Organize imports: external libraries → internal components/hooks → styles/assets.
- ESLint/Prettier compliance: no unused variables, exhaustive `useEffect` dependencies.
- This is fully valid for production handoff. TypeScript is not required.

### 15. Error Boundaries & Resilience

- Wrap major route views in React Error Boundaries.
- Route async failures through a global Toast/Snackbar notification channel.
- Loading Skeletons or inline spinners for all async data states.

## Hygiene, Testing & Handoff

### 16. QA & Test Selectors

- Inject `data-testid` attributes on all primary interactive elements, form controls, and major layout sections.

### 17. Dependency & Environment Hygiene

- Clean up unused packages from `package.json`. Pin critical dependency versions.
- Provide a `.env.example` with all required environment variables stubbed out.

### 18. File Hygiene & Icon Consolidation

- Delete orphaned files, unused imports, and dead utility blocks.
- **SVG Icon Extraction:** Extract inline SVG paths over 10 lines into separate icon files or map to `lucide-react`.

---

# Next.js Dimension Overrides

When the stack is Next.js, these dimensions replace or extend their standard counterparts. All other dimensions apply unchanged.

### 5. State Management (Next.js)
Same as standard, plus one additional check: flag any `useState`, `useContext`, or `useReducer` used inside a Server Component — state hooks only work in Client Components. Any file using them must have `"use client"` at the top. If it doesn't, flag it plainly:
> "This file is managing interactive state but isn't marked as a client component — the developer will need to fix this before it works."

### 10. Lazy Loading (Next.js)
`React.lazy` is not needed — Next.js handles code splitting automatically per page. Instead, check that heavy components (maps, charts, rich text editors) use `dynamic()` from `next/dynamic` with `{ ssr: false }` where appropriate. Flag any `React.lazy` usage as unnecessary.

### 11. Routing & Navigation (Next.js)
`react-router-dom` is irrelevant. Next.js uses file-based routing. Check:
- Pages live in `/app` (App Router) or `/pages` (Pages Router) — not conditionally rendered in a root component
- Dynamic routes use correct `[param]` and `[[...slug]]` conventions
- Route guards are implemented via `middleware.ts` at the project root, not `<ProtectedRoute />` wrappers
- If the project still uses conditional rendering (`if (page === 'home')`) instead of proper file-based routes, flag it as a handoff issue

### 15. Error Boundaries & Resilience (Next.js)
Instead of manually wrapping routes in Error Boundary components, check for Next.js built-in error files:
- `error.tsx` exists at the appropriate layout level to catch runtime errors
- `not-found.tsx` exists for 404 handling
- `loading.tsx` exists for async suspense states
If these files are missing, flag them as needed — they replace the manual Error Boundary pattern.

### 4 & 7. API Stubs & Backend Markers (Next.js — note only)
The `api.ts` stub pattern still applies for frontend data consumption. However, `// @backend` annotations should specify whether the endpoint is expected as a Next.js API route (`/app/api/resource/route.ts`) or an external backend service. Note this distinction in `BACKEND_CONTRACT.md`.

---

# Common Vibecode Smells

| Smell | Direct Production Fix |
| :--- | :--- |
| **Reinvented UI primitives** | Hand-rolled dropdown, modal, tooltip. Replace with shadcn/Radix equivalent, preserve styling. |
| **Duplicated component variants** | Same button/card built 5 different ways. Consolidate into one shared primitive. |
| **Direct mock data imports in components** | `import { MOCK_DATA }` inside a UI component. Move all data through `api.ts` or `api.js` stubs. |
| **Third-party components fetching directly** | Map loading GeoJSON, chart fetching CSV. All external data must flow through `api.ts`/`api.js`, no exceptions. |
| **Contract-to-Runtime Drift** | Align `domain.ts`/`domain.js` definitions with actual UI runtime usages. |
| **Single 1000+ line components** | Split into container + presentational children while guarding the DOM tree. |
| **Buggy vibe-coded interactions** | Flickering hovers, dropping animations. Flag to developer rather than silently preserve. |
| **"Mostly" CSS variables** | Hardcoded colors, spacing, sizing, or typography in Tailwind classes or inline styles. Swap with design tokens. |
| **Cascading state chains** | 5+ independent `useState` hooks. Move to context or a consolidated reducer. |
| **Raw, multi-line SVGs in JSX** | Extract to standalone icon files or `lucide-react`. |

---

# Severity Scale

Every violation must include a severity and a one-line justification. Use this scale consistently:

| Severity | When to use | Examples |
|----------|-------------|----------|
| **High** | Blocks backend handoff or will cause dev rework/integration failure | Direct `fetch()` in components (dim 4); no `domain.ts`/`domain.js`; god-component 800+ lines; missing `api.ts` stubs; conditional routing instead of router |
| **Medium** | Handoff possible but dev friction, maintenance risk, or partial contract drift | Duplicated button variants (1b); hardcoded spacing/typography (9); missing `@backend` on one stub; 5+ `useState` in one file; reinvented modal without Radix |
| **Low** | Polish, hygiene, or non-blocking quality gaps | Missing `data-testid` on secondary elements; unused imports; console.log; arbitrary Tailwind that could snap to token with <1px shift |

When assigning severity, state **why** in the Issue column: e.g., "High — developer cannot swap API layer without touching 12 components."

---

# Execution Modes

## Audit mode

Listen for commands starting with `/vibe-to-prod audit`. Report violations without modifying files.

Output format must strictly follow [references/audit-checklist.md](references/audit-checklist.md).

**Audit rules:**

0. **Plain language first.** Apply mandatory language enforcement from Step 0 before outputting the report.

1. **Lead with stack detection.** Identify the stack first (React/TS, React/JS, Next.js, or unsupported). If unsupported, stop and follow the redirection response in Step 0. If JavaScript, proceed with all dimensions — do not flag as a blocker. If Next.js, apply the dimension overrides.

2. **Show evidence, not just conclusions.** For every dimension marked failing, include the specific file path, **line number**, or grep output that proves the violation. **Ban duplicate citations:** citing the same file twice as two different evidence entries is padding. Each citation must be a distinct `path:line` or distinct grep match. If two violations share a file, cite different line numbers and explain why each is a separate finding.

3. **Run the grep patterns from audit-checklist.md.** These are required evidence-gathering steps, not optional. Include the output (or a summary of the output) in the audit report. If the environment doesn't support shell execution, run equivalent regex searches and show results.

4. **Flag interactions from code — distinct line references required.** You cannot test animations visually in audit mode. Scan for: Framer Motion configs with extreme spring tensions, CSS transitions with 0ms durations, hover handlers that toggle state without debounce, animation components that depend on frequently-changing state values. Each flagged item must include **`file:line`**, the code pattern found, and your reasoning. Never cite the same `file:line` twice without explaining why it represents two separate issues. If nothing is found, write: `None found after code scan.` — do not skip the section.

5. **Design fidelity in audit mode — no fake "AT RISK".** Do not claim layout or animation fidelity is "AT RISK" without visual testing — you have no browser. Instead report: **"Not evaluated in audit mode (code-only scan). Refactor mode preserves design intent per Core Constraints."** Only flag concrete code-level risks (e.g., DOM wrapper changes proposed, animation config extremes).

6. **Dimension 1 and 1b are separate rows.** Never merge reusability into dimension 1. Always report 1b with its own pass/fail and evidence.

7. **Dimension 4 is all-or-nothing.** If any component — including maps, charts, or third-party visual integrations — fetches data directly instead of through `api.ts`/`api.js`, dimension 4 fails. Do not mark it as partially passing.

8. **Dimension 8 requires both lists.** Reinvented primitives list AND genuine custom list — see mandatory format in dimension 8. Missing either = incomplete.

9. **Apply severity scale.** Every violation row includes High/Medium/Low with justification.

```
/vibe-to-prod audit [file or directory]
```

## Refactor mode (default)

Execute a full 18-dimension pass over the targeted module. Refactor cleanly while preserving design intent. Generate or update `BACKEND_CONTRACT.md` using [references/backend-contract-template.md](references/backend-contract-template.md).

```
/vibe-to-prod refactor [file or directory]
```

## Quick mode

Fast-track structure + types + stubs only. Applies dimensions: 1, 1b, 2, 3, 4, 7, 17, 18.

```
/vibe-to-prod quick [file or directory]
```

## Scope inference

If a user request targets a specific concern ("just fix the state management", "only audit the components"), apply only the relevant dimensions. Do not run a full 18-dimension pass on a scoped request. State which dimensions you're applying at the start of your response.

---

# Target File Layout (Handoff-Ready)

**React + TypeScript (default):**

```
src/
├── data/               # Extracted mock/seed data (never imported directly by components)
├── domain.ts           # Canonical business types (API contracts)
├── api.ts              # Typed async stubs with @backend annotations
├── components/
│   ├── ui/             # Shared reusable primitives (one per pattern — Button, Card, Badge)
│   ├── icons/          # Extracted SVG icon components
│   └── ConfirmDialog.tsx
├── contexts/           # One isolated context provider per domain
├── hooks/              # Shared custom hooks
├── routes/             # react-router setup + guards
├── pages/              # Route-level page components
└── utils/              # Helpers, formatters
.env.example            # Required environment variables (stubbed)
BACKEND_CONTRACT.md     # Generated integration doc for dev team
```

**React + JavaScript:** same structure; use `domain.js` and `api.js` instead of `.ts`.

**Next.js:** use `app/` (App Router) or `pages/` (Pages Router) instead of `routes/` + conditional rendering; add `middleware.ts` for route guards; add `error.tsx`, `loading.tsx`, and `not-found.tsx` where appropriate.