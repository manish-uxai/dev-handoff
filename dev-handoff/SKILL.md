---
name: dev-handoff
description: Transforms high-fidelity designer-built React/TS prototypes into production-ready, frontend-only codebases. Use this skill when a designer wants to hand off their coded prototype to a frontend developer, when someone wants to make their vibe-coded UI "dev-ready", when asked to audit or refactor a React/TS prototype for backend integration, or when the goal is to eliminate UI re-development by treating the prototype as the final production frontend. Triggers on phrases like "handoff to dev", "make this production ready", "clean up my prototype", "prep for backend integration", "frontend handoff", "Audit Codebase" or any mention of sharing a codebase with a frontend developer.
license: MIT
compatibility: Works with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, and other agentskills.io-compatible agents. Requires React/TypeScript frontend projects.
metadata:
  author: dev-handoff
  version: "3.0.0"
  framework: 18-dimension-handoff
---

# Philosophy: Figma-less Direct-to-Code Handoff

Traditional handoff is broken. A designer spends weeks — sometimes months — building a polished, high-fidelity prototype in code. Then they hand over a Figma file or screenshots. The frontend developer spends another 1–2 months rebuilding the exact same UI from scratch. Fidelity degrades. Micro-interactions get lost. Time is wasted twice.

**We break this cycle entirely.**

The designer-builder has already developed the final, high-fidelity production UI. Your job is to treat this prototype codebase as the **actual production frontend** and harden the architecture underneath it — so a frontend engineer can open this project, plug in real APIs, and ship. Zero visual rewrites. Zero animation loss. Zero layout shifts. **1–2 months of UI development saved.**

You are not "cleaning up a throwaway demo." You are making the invisible infrastructure production-ready while leaving everything the user can see and feel completely intact.

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

**This is a hard rule, not guidance.** When the developer connects real APIs, they change only the function body inside `api.ts`. Every component that consumes that data updates automatically — no component-level changes required.

---

# The 18-Dimension Handoff Framework

## Structure & Separation

### 1. Component Architecture & DOM Hierarchy Guard

- Isolate stateful logic from presentational components. Enforce one domain context per provider.
- Break massive "god-components" (400+ lines) into focused, presentational child modules.
- **DOM Hierarchy Guard:** Preserve the exact DOM layout hierarchy when splitting components. Do not introduce redundant wrapper elements or alter CSS display properties (`flex`, `grid`) of parent-child relationships — this instantly breaks style containment and positioning.
- **Reusability pass:** After splitting, scan for duplicated UI patterns across the codebase. Consolidate into shared primitives under `components/ui/`. One component per pattern.

### 2. Clean Data Extraction

- Move all `MOCK_` arrays, seed data, and hardcoded lists to `/src/data/`. No inline arrays in UI components.
- Preserve the exact data shape to prevent rendering errors or layout discrepancies.

### 3. Canonical Domain Types (`domain.ts`)

- Establish a single source of truth for business concepts. Every entity the UI renders must map to a strict TypeScript interface.
- Developers use these types to generate or validate backend API contracts.

### 4. API Contract Stubs (`api.ts`)

- **Hard rule:** No component imports mock data directly. All data flows through typed async functions in `api.ts`.
- Implement typed async functions simulating latency (`await delay(300)`) for all data dependencies.
- **Strict Envelope Rule:** Mock responses must use realistic enterprise response envelopes (e.g., `{ data: T, meta: { total, page } }`) rather than flat arrays.
- **Third-Party Integrations:** Maps, charts, heatmaps — never fetch internally. Consume data via API stubs passed through state/context.
- **`// @backend` annotation format** — every stub must carry this annotation:

```ts
// @backend POST /api/studies/:id/assessments
// Auth: Bearer token (JWT)
// Payload: { assessmentId: string; visitId: string; required: boolean }
// Response: { data: Assessment; meta: { updatedAt: string } }
export async function createAssessment(payload: CreateAssessmentPayload): Promise<ApiResponse<Assessment>> {
  await delay(300);
  return { data: MOCK_ASSESSMENTS[0], meta: { updatedAt: new Date().toISOString() } };
}
```

## Backend Integration & State

### 5. State Management & Reducer Isolation

- Avoid chains of 5+ independent `useState` hooks causing cascading re-renders.
- Consolidate state mutations into a reducer-based state slice with explicit action schemas, history metadata, and optional rollback hooks. Allows developers to hook up Redux or Zustand instantly.

### 6. Read-Only / Role-Based Access Control

- Guarded setters must block data mutation. Use the HTML `inert` attribute to block UI interaction for read-only roles.
- Define roles as a strict union type in `domain.ts`. Apply `inert={isReadOnly}` to interactive form containers to freeze modifications without altering styles.

### 7. Backend Integration Markers & Contract Sync

- Every API stub annotated with `// @backend` as shown in dimension 4.
- Ensure annotations perfectly match types defined in `domain.ts` — zero contract-to-runtime drift.
- After refactoring, generate `BACKEND_CONTRACT.md` using the schema in [references/backend-contract-template.md](references/backend-contract-template.md).

## UI Quality & Performance

### 8. Component Library Compliance (New)

- **Audit all custom UI primitives** (dropdowns, modals, tooltips, tabs, popovers, date pickers, checkboxes, radios).
- Replace reinvented primitives with **shadcn/ui** or **Radix UI** equivalents, preserving the designer's exact visual styling.
- Genuine custom components (novel interactions, product-specific visualizations) are preserved and hardened — not replaced.
- Flag any replaced component in `BACKEND_CONTRACT.md` so the developer knows which primitives have clean swap surfaces.

### 9. CSS Token & Design System Compliance

- Map all colors and spacing to semantic CSS variables (e.g., `--color-status-danger`, `--spacing-container`). No hardcoded hex codes remaining after refactor.
- **Defensive Tailwind Snapping:** Scan for Tailwind arbitrary bracket values (e.g., `p-[17px]`, `bg-[#f3f4f6]`). Snap to nearest standard Tailwind scale only if the visual shift is less than `1px`. Otherwise extract to a semantic CSS variable.

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

### 14. Linting, Formatting & Strict Typing

- Eradicate `any` types. Convert legacy JSDoc typings into strict TypeScript interfaces.
- Organize imports: external libraries → internal components/hooks → styles/assets.
- Strict ESLint/Prettier compliance: no unused variables, exhaustive `useEffect` dependencies.

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

# Common Vibecode Smells

| Smell | Direct Production Fix |
| :--- | :--- |
| **Reinvented UI primitives** | Hand-rolled dropdown, modal, tooltip. Replace with shadcn/Radix equivalent, preserve styling. |
| **Duplicated component variants** | Same button/card built 5 different ways. Consolidate into one shared primitive. |
| **Direct mock data imports in components** | `import { MOCK_DATA }` inside a UI component. Move all data through `api.ts` stubs. |
| **Contract-to-Runtime Drift** | Align `domain.ts` interfaces with actual UI runtime usages. |
| **Single 1000+ line components** | Split into container + presentational children while guarding the DOM tree. |
| **Buggy vibe-coded interactions** | Flickering hovers, dropping animations. Flag to developer rather than silently preserve. |
| **"Mostly" CSS variables** | Hardcoded colors or pixel metrics hiding in Tailwind classes. Swap with design tokens. |
| **Cascading state chains** | 5+ independent `useState` hooks. Move to context or a consolidated reducer. |
| **Raw, multi-line SVGs in JSX** | Extract to standalone icon files or `lucide-react`. |

---

# Execution Modes

## Audit mode

Listen for commands starting with `/dev-handoff audit`. Report violations without modifying files.

Output format must strictly follow [references/audit-checklist.md](references/audit-checklist.md).

```
/dev-handoff audit [file or directory]
```

## Refactor mode (default)

Execute a full 18-dimension pass over the targeted module. Refactor cleanly while preserving design intent. Generate or update `BACKEND_CONTRACT.md` using [references/backend-contract-template.md](references/backend-contract-template.md).

```
/dev-handoff refactor [file or directory]
```

## Quick mode

Fast-track structure + types + stubs only. Applies dimensions: 1, 2, 3, 4, 7, 17, 18.

```
/dev-handoff quick [file or directory]
```

## Scope inference

If a user request targets a specific concern ("just fix the state management", "only audit the components"), apply only the relevant dimensions. Do not run a full 18-dimension pass on a scoped request. State which dimensions you're applying at the start of your response.

---

# Target File Layout (Handoff-Ready)

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