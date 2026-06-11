---
name: vibe-to-prod
description: Transforms high-fidelity designer-built React or Next.js prototypes into production-ready codebases. Detects the project stack and adapts automatically. The output is clean, production-survivable code — not a documentation package. Use this skill when a designer wants to make their vibe-coded UI production-ready, when someone says "make this dev-ready", "clean up my prototype", "prep for integration", "vibe to prod", or any mention of turning a coded prototype into something a developer can ship.
license: MIT
compatibility: Works with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, and other agentskills.io-compatible agents. Supports React and Next.js projects. JavaScript codebases are migrated to TypeScript automatically — output is always TypeScript. Other stacks trigger guided redirection.
metadata:
  author: vibe-to-prod
  version: "5.0.0"
  framework: 21-dimension-handoff
---

# Philosophy

Traditional handoff is broken. A designer spends weeks building a polished, high-fidelity prototype in code. Then they hand over a Figma file or screenshots. The frontend developer spends another 1–2 months rebuilding the exact same UI from scratch. Fidelity degrades. Micro-interactions get lost. Time is wasted twice.

**We break this cycle entirely.**

The designer-builder has already developed the final production UI. Your job is to harden the code so it survives real data, real users, and real edge cases — then a developer opens the project and starts integrating APIs immediately. **1–2 months of UI development saved.**

**The output is production-ready code, not a documentation package.** No separate contract files. No lists of what was changed. The code itself is the deliverable — clean enough that a developer can read it and understand everything they need.

---

# Pre-flight: Node.js check

**Before anything else — before stack detection, before any mode runs — verify Node.js is installed.** Run `node --version` as the very first command.

**If it succeeds:** note the version silently and continue to Step 0.

**If it fails:** stop all code operations immediately. Do not attempt npm install, skill installation, scaffolding, or file edits — nothing works without Node, and partial work the designer can't run or test is worse than a clean stop. Tell the designer plainly:

> "Node.js isn't installed on your machine yet. It's needed to run React projects and install dependencies. Here's how to set it up:
>
> **Mac:** Open Terminal and run `brew install node` (if you have Homebrew) or download from https://nodejs.org
> **Windows:** Download the installer from https://nodejs.org and run it — accept all defaults.
>
> Once installed, close and reopen VS Code, then run this skill again. Everything will work from there."

Do not attempt to do the rest of the work without Node. A clear stop with a clear next step is better than a half-broken state.

---

# Step 0: Stack Detection

**Identify what the user has built.** Check `package.json`, file extensions, and folder structure.

---

## React (with TypeScript) — `.tsx` / `.ts`

Proceed with all 21 dimensions.

---

## React (with JavaScript) — `.jsx` / `.js`

**vibe-to-prod always outputs TypeScript.** A JavaScript codebase is migrated to TypeScript as the mandatory first step — this is not optional and the user is not asked. TypeScript is the production-handoff standard: it gives the receiving developer real type contracts, autocomplete, and refactoring safety that JavaScript can't.

Tell the user plainly, then proceed:

> "Your project is in JavaScript. I'll convert it to TypeScript first — this gives your developer proper type safety and makes the handoff cleaner. Then I'll run the full production pass on the TypeScript version."

Read and follow `references/jsx-to-tsx-migration.md` to migrate, THEN run the 21 dimensions on the resulting TypeScript codebase. Because migration happens first, everything downstream is TypeScript — there is no JavaScript path through the dimensions, and PropTypes are never used (TypeScript interfaces replace them entirely).

---

## Next.js — `next.config.*` present or `next` in `package.json`

Proceed with all 21 dimensions, applying the Next.js overrides listed later in this document. If the Next.js project is in JavaScript, migrate to TypeScript first, same as above.

---

## Plain HTML / CSS / JS — no `package.json`, or only `.html` / `.css` files

The project isn't React yet, but the UI work has value. vibe-to-prod can convert it to a production-ready Vite + React + TypeScript project — the HTML becomes React components, inline styles become design tokens, and the result goes through the full 21-dimension pass.

Tell the designer:

> "Your project is built with plain HTML and CSS. I'll convert it to a React + TypeScript project first — your visual design stays the same, but it gets proper components, data flow, and production structure. Then I'll run the full production pass so a developer can start integrating immediately."

**Conversion flow:**

1. Scaffold a Vite + React + TypeScript project (Path A from `references/scaffold.md`)
2. Convert HTML to React — read and follow `references/html-to-react.md`
3. Run the 21 dimensions on the resulting React codebase

---

## Vue.js, Svelte, Angular, or other frameworks

Do not proceed. Explain:

> "Your project uses [detected framework]. This skill is built around React and Next.js, so running it here would give inaccurate results.
>
> Two options: I can help migrate to React so the full pipeline works, or I can give general handoff advice without the structured audit. Which works better?"

---

## Language and tone rule

**This skill is used by designers and PMs growing into design engineers, as well as developers.** The audit serves both: a designer should be able to read it and learn, a developer should be able to act on it.

**Findings: technical term + plain consequence.**
Keep the precise technical term (it's how a designer learns the vocabulary), but pair it with a plain clause explaining why it matters. The term teaches; the consequence clarifies.

- Good: "`any` types in 12 components — a developer gets no autocomplete or type safety, so integration bugs surface at runtime instead of compile time."
- Good: "Direct fetch in PopulationMapView.tsx:42 — bypasses the API layer, so swapping in a real backend later means rewriting the component."
- Avoid: "Loose typing." (term with no teaching consequence)
- Avoid: "Components don't say what they need." (consequence with no term — designer learns nothing)

**Exceptions — always full, clear prose (never compressed):**

- Security warnings (hardcoded keys, XSS risks)
- Irreversible or destructive actions
- Anything design-system or design-related (this is the technical area designers engage with most — explain it fully)

**Summary: warm and plain.**
The end-of-audit summary is the one section written purely for a non-technical reader. No jargon. Covers overall health, the top 3 things to fix, and what it means for handing off to a developer. This is where the translation table applies.

| Technical term (in summary, translate to) | Plain version                                |
| :---------------------------------------- | :------------------------------------------- |
| god-component                             | one file doing too many things               |
| API stubs / data layer                    | where real data will plug in                 |
| design tokens                             | reusable color and spacing values            |
| error/empty states                        | what users see when data fails or is missing |
| PropTypes / types                         | components describing what data they expect  |
| routing                                   | how the app moves between pages              |
| hardcoded secret                          | a password or key exposed in the code        |

Read the user's messages for cues on how technical to make the summary. If they use "component", "props", "state" correctly, the summary can carry a bit more vocabulary.

---

## Severity scale

Every finding must include a severity level with this definition:

| Severity   | Meaning                                                                                  |
| :--------- | :--------------------------------------------------------------------------------------- |
| **High**   | Will break in production, block API integration, or cause data loss. Fix before handoff. |
| **Medium** | Creates friction for the developer or degrades user experience. Fix soon after handoff.  |
| **Low**    | Cleanup that improves code quality. Can be addressed over time.                          |

---

# Core Constraints

## 1. Preserve Design Intent, Allow Implementation Improvements

The **design surface** — layout, spacing, visual hierarchy, interaction behavior, animation feel — is sacred.

The **engineering substrate** — how animations are implemented, how state flows, how components are structured — can and should be improved, as long as the perceived result is indistinguishable or better.

**When a vibe-coded interaction appears buggy rather than intentional, flag it rather than silently preserving or fixing it.**

**Code-detectable interaction smells (no browser required):**

- Framer Motion springs with `stiffness` > 500 or `damping` < 5 (likely jank)
- CSS transitions with `0ms` or `0s` durations (likely missing)
- Hover/focus handlers toggling state without debounce (likely flicker)
- Animation components depending on frequently re-rendering parent state (likely frame drops)
- `setTimeout`/`setInterval` driving visual transitions instead of CSS or animation libraries

## 2. Component Strategy

Vibe-coded prototypes hand-roll UI primitives — dropdowns, modals, tooltips, tabs — that look correct in demos but lack keyboard navigation, focus trapping, scroll-locking, and edge-case resilience. They break in production with real users.

**Replace reinvented primitives** with **shadcn/ui** or **Radix UI** equivalents, preserving the designer's exact visual styling. These are production-tested and accessible — a developer can ship them as-is without needing their own design system. If the developer does have a design system later, shadcn components are easy to swap prop-for-prop.

**Consolidate duplicated components.** Scan the entire codebase for the same button, card, badge, input, or layout block rebuilt with slight variations across files. Merge into shared primitives under `components/ui/`.

**Preserve genuine custom components** — novel interactions or product-specific visualizations that no library offers. These ship as-is.

## 3. API Stubs Are Mandatory — No Direct Data in Components

Components must never import mock data arrays directly or call `fetch()` internally. All data must flow through async functions in `api.ts`.

**This applies to every component without exception** — maps fetching GeoJSON, charts loading CSV/JSON, tables pulling datasets. If a component gets data from anywhere other than a prop or an API stub function, it violates this rule.

When the developer connects real APIs, they change only the function body inside `api.ts`. Every component updates automatically.

---

# The 21-Dimension Framework

## Structure & Separation

### 1. Component Architecture & DOM Hierarchy Guard

- Isolate stateful logic from presentational components.
- Break god-components (400+ lines) into focused child modules.
- **DOM Hierarchy Guard:** Preserve the exact DOM layout hierarchy when splitting. No redundant wrappers or altered CSS display properties.
- **Circular dependency check:** Scan for components importing each other in a loop. Circular imports break builds and cause subtle bugs. Common in vibe-coded prototypes where the AI generates imports without considering the dependency graph. In audit mode, flag any circular chain found.
- **Absolute path aliases:** Enforce `@/*` path mapping across the codebase. No `../../` relative imports — they break when files move and make the codebase harder to navigate. Configure in `tsconfig.json` (TS) or `jsconfig.json` (JS) and `vite.config.ts`. In audit mode, flag any import using `../..` as a violation.
- **Reusability pass (separate audit step):** Scan the entire codebase — not just the audited module — for duplicated UI patterns. Practical detection methods:
  - Search for similarly named component files across folders (e.g., `Button.jsx` in three different directories)
  - Search for components with overlapping `className` patterns (long identical class strings across files)
  - Search for copy-pasted JSX structures: similar element trees with minor prop differences
  - Read the `components/` directory structure and flag files that serve the same purpose
  - In audit mode, list every duplicated pattern with file locations. This is not optional.

### 2. Clean Data Extraction

- Move all `MOCK_` arrays, seed data, and hardcoded lists to `/src/data/`. No inline arrays in UI components.
- Preserve the exact data shape.

### 3. Canonical Domain Types (`domain.ts`) + Runtime Validation

A single file that describes every data entity the UI renders. The codebase is TypeScript (migrated first if it arrived as JS), so this is always strict interfaces — never JSDoc.

Strict interfaces in `domain.ts`:

```ts
export interface Patient {
  id: string;
  name: string;
  status: "active" | "inactive";
}
```

**Quality check:** No duplicated or conflicting interface definitions across the codebase. Every interface must be complete — all fields the UI actually uses must be typed. In audit mode, verify interfaces match actual runtime usage.

**Runtime validation (catches bad API responses before they crash the UI):**

Check `package.json` for an existing validation library first:

- If **Zod** is present — add Zod schemas in `src/schemas/` alongside domain types
- If **Yup** is present — declare schemas using `yup.object()` and extract types with `yup.InferType<typeof Schema>`
- If **Valibot** or **Joi** is present — use that library's equivalent schema pattern
- If **nothing** is present — add Zod as the default: `npm install zod`

The principle is the same regardless of library: every API response shape should be validated at runtime so bad data from a real API is caught at the boundary, not deep inside a component.

```ts
// Example with Zod
import { z } from "zod";

export const PatientSchema = z.object({
  id: z.string(),
  name: z.string(),
  status: z.enum(["active", "inactive"]),
});

export type Patient = z.infer<typeof PatientSchema>;
```

### 4. API Contract Stubs (`api.ts`) + Data Fetching Layer

- **Hard rule:** No component imports mock data directly. All data flows through async functions in `api.ts`. This rule is non-negotiable regardless of the data fetching library in use.
- Implement async functions simulating latency (`await delay(300)`) for all data dependencies.
- **Envelope Rule:** Mock responses use realistic envelopes (`{ data, meta: { total, page } }`) not flat arrays.
- **Third-Party Integrations:** Maps, charts, heatmaps — never fetch internally. Data via API stubs only.
- Every stub carries a `// @backend` annotation.

**Data fetching layer (wraps the API stubs with loading/error/caching):**

Check `package.json` for an existing data fetching library first:

- If **TanStack Query** (React Query) is present — wrap API stubs in `useQuery`/`useMutation` hooks inside `src/api/hooks/`
- If **SWR** is present — wrap in custom `useSWR` functions using the stub as the fetcher
- If **Axios with custom hooks** is present — wrap in that pattern
- If **nothing** is present — add TanStack Query as the default: `npm install @tanstack/react-query`

TanStack Query gives loading, error, refetch, and caching states automatically — components get these for free instead of managing them manually with `useState`.

```ts
// src/api/hooks/usePatients.ts
import { useQuery } from "@tanstack/react-query";
import { getPatients } from "../api";

export function usePatients() {
  return useQuery({
    queryKey: ["patients"],
    queryFn: getPatients,
  });
}
```

Components consume the hook, never the raw API function or mock data directly:

```tsx
const { data, isLoading, error } = usePatients();
```

**TypeScript annotation format:**

```ts
// @backend GET /api/patients
// Auth: Bearer token (JWT)
// Response: { data: Patient[]; meta: { total: number } }
export async function getPatients(): Promise<ApiResponse<Patient[]>> {
  await delay(300);
  return { data: MOCK_PATIENTS, meta: { total: MOCK_PATIENTS.length } };
}
```

(Codebase is always TypeScript, so annotations live above typed stubs as shown above.)

## State & Access

### 5. State Management & Reducer Isolation

- Avoid chains of 5+ independent `useState` hooks.
- Consolidate into reducer-based state with explicit action schemas. Allows developers to hook up Redux or Zustand instantly.

### 6. Read-Only / Role-Based Access Control

- Use the HTML `inert` attribute to block UI interaction for read-only roles.
- Define roles as a union type in `domain.ts`.

### 7. Backend Integration Markers

- Every API stub annotated with `// @backend` as shown in dimension 4.
- Annotations must match data shapes in `domain.ts`.
- The annotations live in the code — a developer reading `api.ts` can see every endpoint they need to build.

## UI Quality & Performance

### 8. Component Library Compliance

- **Actively scan** for reinvented UI primitives (dropdowns, modals, tooltips, tabs, popovers, date pickers, select menus).
- Replace with **shadcn/ui** or **Radix UI** equivalents, preserving visual styling.
- Preserve genuine custom components.
- **Do not auto-pass this dimension.** If no scan was performed, mark as unevaluated.

### 9. Design Token Compliance

Covers **colors, spacing, sizing, typography, and all visual values.** Not just colors.

- Map all colors to semantic CSS variables (`--color-status-danger`). No hardcoded hex codes.
- Map spacing and sizing to design tokens or standard Tailwind scale. No raw pixel values in inline styles.
- **Scan for all hardcoded visual values:**
  - Hex colors in JSX and CSS
  - Tailwind arbitrary bracket values (`p-[17px]`, `bg-[#f3f4f6]`, `w-[243px]`, `gap-[12px]`)
  - Inline style raw pixels (`style={{ padding: '12px', fontSize: '14px', width: '240px' }}`)
  - CSS file raw pixels (`margin: 16px`, `font-size: 13px`, `border-radius: 8px`)
- **Defensive Tailwind Snapping:** Snap arbitrary values to nearest standard Tailwind scale only if visual shift < 1px. Otherwise extract to a semantic CSS variable.

### 10. Destructive Action Safety

- Gate all deletions, resets, or irreversible state changes behind `ConfirmDialog` or "Toast with Undo."

### 11. Routing & Navigation (`react-router`)

- Replace conditional page rendering with declarative `react-router-dom` paths.
- Implement lazy loading (`React.lazy`) for heavy route containers.
- Set up Route Guards for authenticated/role-based views.

### 12. Component Performance

- `useMemo` for expensive filtering, sorting, calculations.
- `useCallback` for event handlers passed to children.
- `React.memo` on heavy list items, cards, or data grid rows.

## Robustness & Production Readiness

### 13. Accessibility (a11y) Baseline

- Eliminate "div soup." Use semantic HTML5 (`<nav>`, `<main>`, `<header>`, `<button>`, `<aside>`).
- Interactive elements need `aria-label` or `aria-labelledby`.
- Keyboard focus: `tabIndex`, visible focus states, focus trapping in modals.
- `aria-live` regions for dynamic updates.

### 14. Linting, Formatting & Code Quality

The codebase is TypeScript (migrated first if it arrived as JS), so:

- Eradicate `any` types. Strict interfaces everywhere.
- Organized imports. ESLint/Prettier compliance.
- No PropTypes — ever. TypeScript interfaces are the type contracts; PropTypes are redundant and must not be added. If migrating from JS, PropTypes are removed and replaced by interfaces, not kept alongside them.

### 15. Error Boundaries & Component Resilience

**Route-level:**

- Wrap major views in React Error Boundaries.
- Global Toast/Snackbar for async failures.

**Component-level (every data-consuming component must handle):**

- **Loading state:** what the user sees while data loads (skeleton, spinner, or placeholder — not blank space)
- **Error state:** what happens when the data call fails (fallback message — not a crash)
- **Empty state:** what appears when there's no data (helpful message — not invisible component)

### 16. QA: Test Selectors & Test Existence

**Test selectors:**

- `data-testid` on all primary interactive elements, form controls, and major layout sections.

**Test existence:**

- Check if any test files exist (`*.test.*`, `*.spec.*`, `__tests__/` folders)
- Check if a test runner is configured in `package.json` (vitest, jest, playwright, cypress)
- If no tests exist at all, flag as Medium severity — a developer inherits a codebase with no safety net for refactoring
- If tests exist, note what's covered and what's not. Don't audit test quality — just confirm the safety net exists.

### 17. Dependency, Environment & Onboarding Hygiene

- Clean unused packages. Pin critical dependency versions.
- Provide `.env.example` with all required environment variables stubbed.
- **README setup check:** A developer cloning this repo should be able to run the app within 5 minutes. Verify:
  - `README.md` exists and is not the default template
  - It includes install instructions (`npm install`)
  - It includes run instructions (`npm run dev`)
  - It mentions any required environment variables
  - If README is missing or only a default template, flag as Medium severity

### 18. File Hygiene & Icon Consolidation

- Delete orphaned files, unused imports, dead code.
- **Icon replacement (lookup first, extract last):**
  1. Find all inline SVGs in JSX components
  2. For each SVG, identify what it represents — use file name, component name, and surrounding code as context if the path is unclear
  3. Check **lucide-react** first (1500+ icons, most prototype icons have a match)
  4. If not found, check **heroicons** or **phosphor-icons**
  5. If a library equivalent exists — replace the entire SVG block with a single import line
  6. Only extract to a custom icon file in `components/icons/` if no reasonable library equivalent exists
  7. Never leave raw multi-line SVG coordinate paths inside UI components

### 19. Component Production Readiness

This dimension checks whether components survive real-world conditions, not just the prototype's mock data.

**Data resilience:**

- **Null/undefined fields:** component renders gracefully when a data field is missing, not crashes
- **Text overflow:** long names, descriptions, URLs — truncated or wrapped, not breaking the layout
- **Variable data lengths:** component works with 0 items, 1 item, 500 items — not just the 3 mock items
- **Special characters:** component doesn't break on quotes, ampersands, unicode

**Reusability:**

- **Baked-in UI text:** labels, placeholders, button text, error messages baked directly into a component instead of passed as props. (Note: this is UI _strings_ — distinct from dimension 2, which covers hardcoded _data arrays_.) Baked-in text reduces reusability — flag as Low.
- **Prop interface clarity:** TypeScript interfaces define what the component accepts. A developer should understand the component's API without reading its internals.
- **Style isolation:** component works regardless of parent context. Doesn't depend on a specific wrapper's width or CSS for its own layout.

**Responsive basics:**

- No fixed pixel widths that prevent the component from adapting
- If responsive behavior is explicitly out of scope for the project, note it and move on

**Coverage method (scan all, deep-read capped):**

- Run the null-access and fixed-width grep patterns across ALL components (cheap — grep returns matching lines, not whole files)
- From the flagged candidates, deep-read the highest-risk ones, capped at 8 components maximum
- Report honestly: "grep flagged 23 components with potential null-access; deep-read the 8 highest-risk; recommend reviewing the remaining 15"
- Never report "passed" based on a small sample — always state how many were flagged vs how many were deep-read

### 20. Security Basics

Frontend prototypes often contain security holes that get inherited by the developer. Check for:

**Hardcoded secrets:**

- API keys, tokens, or credentials in source code (not just `.env` — check actual JS/TS files)
- Firebase configs, Stripe keys, auth tokens written directly in code
- Any string that looks like a key: long alphanumeric strings, strings starting with `sk_`, `pk_`, `AKIA`, `ghp_`

**Dangerous patterns:**

- `dangerouslySetInnerHTML` — XSS risk if used with user-provided data
- Unvalidated URL parameters used in fetch calls or link hrefs
- `eval()` or `new Function()` usage

**Committed secrets:**

- `.env` file (not `.env.example`) committed to git — check `.gitignore`
- Any file containing real API endpoints with keys in query parameters

**Dependency vulnerabilities:**

- Run `npm audit` — flag any high or critical severity vulnerabilities

**In audit mode:** flag any hardcoded secret as High severity — this is a real security risk, not a code quality issue. Flag `dangerouslySetInnerHTML` as Medium unless it's used with sanitized content.

### 21. Design Quality (No AI Slop)

This is the dimension designers care about most — it protects visual craft, not just code structure. Vibe-coded UIs drift toward generic "AI dashboard" tropes and inconsistent styling. Check for and flag:

**Avoid:**

- Decorative accent borders on cards (thick `border-left`/`border-right` to imply importance) — generic AI-dashboard cliché
- Random or one-off colors not defined as semantic tokens — every color should map to a token in `:root`
- Inline `style={{ color: '#…' }}` or ad-hoc hex/rgb in components
- Turning every data series the same alert color (e.g. all bars red when "critical") — this destroys metric identity

**Prefer:**

- Semantic tokens for everything: `--color-category-*`, `--color-status-*`, `--color-fg-muted`
- Legends that match what's drawn — color encodes the metric type, severity is a separate visual cue
- Tooltips on flagged chart elements explaining what the metric is, its value, and why it matters
- shadcn `Tooltip`, `Badge`, `Card` without custom chrome borders

**Severity vs metric color (important for data viz):**
Keep two encoding layers separate:

- **Fill color = which metric** (patient = one token, site = another, cost = another)
- **Ring / badge / icon = severity** (warning or critical outline)

Never recolor an entire chart series red to signal severity — the viewer loses track of which series is which. Severity is an overlay, not a replacement for metric identity.

**In audit mode:** these are mostly Medium or Low severity (visual quality, not blockers), but they're the findings a designer most wants to see. Explain them clearly and fully — this is a design-system area, so never compress these findings.

---

# Next.js Dimension Overrides

When the stack is Next.js, these replace their standard counterparts. All other dimensions apply unchanged.

### 5. State Management (Next.js)

Same as standard, plus: flag any `useState`, `useContext`, or `useReducer` inside a Server Component. State hooks only work in Client Components — files using them need `"use client"` at the top.

### 10. Lazy Loading (Next.js)

`React.lazy` not needed. Check heavy components use `dynamic()` from `next/dynamic` with `{ ssr: false }`.

### 11. Routing & Navigation (Next.js)

`react-router-dom` is irrelevant. Check:

- Pages in `/app` (App Router) or `/pages` (Pages Router)
- Dynamic routes use `[param]` and `[[...slug]]` conventions
- Route guards via `middleware.ts`, not `<ProtectedRoute />` wrappers
- No conditional rendering (`if (page === 'home')`) instead of file-based routes

### 15. Error Boundaries & Resilience (Next.js)

Check for built-in error files instead of manual Error Boundaries:

- `error.tsx` at appropriate layout levels
- `not-found.tsx` for 404 handling
- `loading.tsx` for suspense states
  Component-level resilience checks (loading/error/empty states) still apply.

### 4 & 7. API Stubs & Backend Markers (Next.js)

The `api.ts` stub pattern still applies. `// @backend` annotations should note whether endpoints are expected as Next.js API routes (`/app/api/resource/route.ts`) or external services.

---

# Common Vibecode Smells

| Smell                                        | Fix                                                                                                                                                 |
| :------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Reinvented UI primitives**                 | Replace with shadcn/Radix, preserve styling                                                                                                         |
| **Duplicated component variants**            | Consolidate into one shared primitive                                                                                                               |
| **Copy-pasted layouts**                      | Extract repeated page sections into shared components                                                                                               |
| **Direct mock data in components**           | Route all data through `api.ts` stubs                                                                                                               |
| **Third-party components fetching directly** | Maps, charts fetching their own data — route through API stubs                                                                                      |
| **No empty/error/loading states**            | Component shows blank or crashes with no data — add fallbacks                                                                                       |
| **Hardcoded content in components**          | Labels, placeholders baked in — pass as props instead                                                                                               |
| **Hardcoded visual values**                  | Raw pixels, hex colors in inline styles or arbitrary Tailwind — use design tokens                                                                   |
| **God-components (400+ lines)**              | Split into container + children, guard the DOM hierarchy                                                                                            |
| **Cascading useState chains**                | 5+ hooks — consolidate into reducer or context                                                                                                      |
| **Components crash on null data**            | Add defensive checks for missing fields                                                                                                             |
| **Fixed widths breaking layout**             | Use relative sizing or constrained max-widths                                                                                                       |
| **Raw multi-line SVGs in JSX**               | Check lucide-react first, then heroicons/phosphor. Replace with library import if match found. Only extract to custom icon file if no match exists. |
| **Hardcoded API key in source**              | Move to `.env` and add `.env` to `.gitignore`. Never commit real keys.                                                                              |
| **dangerouslySetInnerHTML**                  | Replace with safe rendering or sanitize input with DOMPurify.                                                                                       |
| **No tests exist**                           | A dev inherits no safety net. At minimum configure a test runner and add tests to critical paths.                                                   |
| **README missing setup instructions**        | A dev can't run the app without asking the designer. Add install + run steps.                                                                       |
| **Circular imports**                         | Components importing each other in a loop. Breaks builds and causes subtle bugs.                                                                    |

---

# Execution Modes

**Routing the command:**

- `/vibe-to-prod start` → start mode (greenfield, installs skills + scaffolds + writes guidelines)
- `/vibe-to-prod scaffold` → scaffold mode (greenfield setup only)
- `/vibe-to-prod audit` → audit mode (report only, no changes)
- `/vibe-to-prod refactor` or `/vibe-to-prod` with existing code → refactor mode (default)
- `/vibe-to-prod quick` → quick mode (structure + types + stubs only)

**Bare `/vibe-to-prod` on an existing codebase = refactor mode.**

**Always write `guidelines.md` and `design.md` to the project root, in every mode.** Whether the user runs start, scaffold, audit, refactor, or quick — ensure the project has both files. If missing, create them. If they exist, merge in anything missing. guidelines.md is the code convention layer; design.md is the design system source of truth (colors, typography, spacing, component rules — in designer language, not code). The agent reads design.md to generate and update the theme CSS tokens. In audit mode (which otherwise makes no changes), writing these files is the one permitted addition — note it in the report.

## Audit mode

Listen for `/vibe-to-prod audit`. Report violations without modifying files.

Output format must follow [references/audit-checklist.md](references/audit-checklist.md).

**Audit rules:**

1. **Read before judging.** Before running any grep patterns, spend 2 minutes mapping the project: what is it for, what stack, what conventions are already in place? This prevents false positives — you won't flag JSDoc as a problem if you first understand it's a JS codebase by design.

2. **Lead with stack detection.** Identify the stack (React/TS, React/JS, Next.js, or unsupported). If unsupported, stop and follow Step 0 redirection. If Next.js, apply overrides. **If the codebase is JSX/JS, note at the top of the report that it will be migrated to TypeScript as the first refactor step** — don't audit it as a permanent JavaScript project, and don't flag "missing PropTypes" as a finding (PropTypes won't exist post-migration; TS interfaces replace them).

3. **Prefer 15 high-confidence findings over 50 speculative ones.** If you can't point to a specific file and line to prove a finding, don't include it. Speculative findings waste the reader's time and undermine trust in the real findings.

4. **Label facts vs judgments.** A fact is: "this function has no error handling: api.js:142." A judgment is: "this module's responsibilities feel unclear." Both are valid — but label which is which so the reader knows what's proven and what's an opinion.

5. **Show evidence, not conclusions.** Every failing dimension must cite specific file paths WITH line numbers.

   **Citation format rules (non-negotiable):**
   - Every citation must include a line number: `api.js:42` not just `api.js`
   - Citing the same file multiple times is ONLY acceptable if each citation has a DIFFERENT line number and describes a DIFFERENT issue
   - If you catch yourself writing `api.js, api.js, api.js` — STOP. That is padding. Write `api.js:42 (missing @backend on getPatients), api.js:67 (missing @backend on createPatient), api.js:91 (flat response instead of envelope)` instead.
   - If you cannot provide a line number, write "file:? (line not verified)" — this is honest and acceptable. Writing the same filename three times is not.

   **WRONG:** `Evidence: domain.js, domain.js, domain.js`
   **WRONG:** `Evidence: SoaWorkspace.jsx, SoaWorkspace.jsx`
   **RIGHT:** `Evidence: domain.js:45 (duplicate PopulationView typedef), domain.js:112 (conflicting Scenario fields)`
   **RIGHT:** `Evidence: SoaWorkspace.jsx:134 (delete handler), SoaWorkspace.jsx:201 (reset handler)`

   **Verify numbers before stating them.** If you report a file has 23,053 lines or a context has 26 useState calls — that number must come from an actual grep/awk output you ran, not from estimation. If you didn't run the count command, don't state a specific number. Say "very large file" or "many useState calls" instead. A wrong number destroys the credibility of the entire report.

6. **Run the grep patterns from audit-checklist.md.** Required, not optional. Include output summaries in the report.

7. **Flag interactions from code.** Scan for extreme spring configs, zero-duration transitions, undebounced hover handlers, setTimeout-driven visuals. List findings with file + line, or explicitly state "No interaction smells found." Never skip this section.

8. **Dimension 4 is all-or-nothing.** Any component fetching directly = fail.

9. **Dimension 8 requires active scanning.** Search for reinvented primitives using the grep patterns in audit-checklist.md. If no scan was performed, mark as unevaluated — not passing.

10. **Dimension 19 coverage.** Run null-access and fixed-width greps across ALL components. Deep-read the highest-risk flagged candidates, capped at 8. Report how many were flagged vs deep-read — never report "passed" on a small sample.

11. **Dimension 20 requires security scanning.** Run the security grep patterns. Flag any hardcoded secret as High immediately.

12. **Dimension 21 (design quality) — explain fully, never compress.** This is the designer's home turf. Flag AI-slop patterns, color-token drift, and severity-vs-metric color issues in full clear prose. These are usually Medium/Low but matter most to the audience.

13. **Every finding must include a severity (High/Medium/Low) with justification.** Not just the label — one sentence explaining why.

14. **Trust grep output — don't over-read files.** The grep evidence pack proves most findings on its own. Only deep-read a source file when the finding genuinely requires seeing code structure (dimension 19 null-handling, confirming a god-component's internal organization). Reading files to "double-check" a finding grep already proved wastes tokens. Cap verification reads to what's strictly necessary.

15. **Design fidelity note:** In audit mode, visual fidelity cannot be verified without running the app. State this once at the top: "Visual fidelity should be verified in the browser after any refactoring." Do not mark it as "AT RISK."

16. **End with a "Summary for the designer."** A warm, plain-language section (no jargon — use the translation table) covering:
    - Overall health in a sentence
    - Top 3 things to fix and why they matter for handoff
    - Roughly how much developer time the fixes will save
    - **Design system status:** is design.md present? Are the tokens consistent? Any color/spacing drift? This is the part designers care about most — tell them whether their design system is being respected or drifting
    - **Component library status:** are shadcn components being used, or are there hand-rolled alternatives?
    - **Visual fidelity note:** remind them to verify in the browser after any refactoring

    This is the part a non-technical reader actually reads. It should make the designer feel informed and in control.

17. **Self-review gate before finalizing.** Re-read your own report and verify each of these before sending. If any fail, fix before finalizing:
    - Every citation has a line number (or honest `file:? (line not verified)`)
    - No filename appears multiple times without distinct line numbers and distinct issues
    - Every specific number (line counts, hook counts) came from actual command output, not estimation
    - Every finding pairs a technical term with a plain consequence
    - Security and design-system findings are in full clear prose, not compressed
    - The designer summary contains no untranslated jargon

```
/vibe-to-prod audit [file or directory]
```

## Refactor mode (default)

Full 21-dimension pass. Refactor while preserving design intent. The `// @backend` annotations in `api.ts` are the only integration contract — no separate document is generated.

**Pre-flight:** Run `node --version` first. If Node.js is missing, stop and direct the designer to install it (see pre-flight section at top of this file).

**If the codebase is plain HTML (no React):** don't refuse — convert it. Scaffold a Vite + React + TypeScript project first (Path A from `references/scaffold.md`), convert the HTML files into React components (see Step 0 HTML detection for the conversion flow), then continue with the 21 dimensions on the resulting React codebase.

**Step zero — TypeScript migration (if needed):** If the codebase is JSX/JS (React but not TypeScript), migrate it to TypeScript FIRST, before any other dimension work, by reading and following `references/jsx-to-tsx-migration.md`. This is automatic and not asked. Everything downstream assumes TypeScript. Migrating first means PropTypes is never added — TypeScript interfaces are the type contracts. Never add PropTypes to a JS file you're about to migrate; convert it instead.

**Avoid over-engineering. When the user says "fix it all," that does not mean apply every possible change maximally. Use judgment:**

- Split god-components, but start with data-bound ones — don't decompose every large file in one pass; some are fine as-is
- Don't globally remove every fixed pixel width — target ones that genuinely break responsive layout, leave acceptable micro-constraints
- Don't migrate routing before fixing data flow — data boundary cleanup gives bigger handoff value first
- Don't add memoization everywhere — only where it measurably helps
- The goal is a codebase a developer can integrate, not a codebase that satisfies every linter rule at the cost of churn

**High-volume cleanup — do the important ones, then offer to continue.** For dimensions that touch many files (data-testid across all components, icon extraction, splitting multiple large files), fix the most important instances first, then ask: "I've done the key ones — want me to continue across the remaining N files? That's thorough but uses more credits." Let the user opt into the long tail rather than exhaustively processing every file by default. This keeps a single run affordable.

**Batch validation — don't validate after every small edit.** Group related edits and run `test && build` at meaningful checkpoints (after finishing a dimension or a logical group), not after every 3-5 file batch. Repeated per-batch validation multiplies cost for little added safety.

```
/vibe-to-prod refactor [file or directory]
```

## Start mode (greenfield — zero decisions)

Trigger: `/vibe-to-prod start` (or the user says "start", "new project", "set me up").

For a designer starting completely fresh. No questions about stack — the answer is always **Vite + React + TypeScript**. The decision is made for them so there's zero friction.

**Pre-flight:** Run `node --version` first. If Node.js is missing, stop and direct the designer to install it (see pre-flight section at top of this file). Everything in start mode — skill installation, scaffolding, dev server — requires Node. If a skill install fails for other reasons (network, repo unavailable), note which one failed and continue with the rest rather than halting.

**Step 1 — install the companion skills.** These give the agent best-practice knowledge for the build. Run each (the `-y` flag skips prompts):

```bash
npx skills add jakubkrehel/make-interfaces-feel-better -y
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices -y
npx skills add https://github.com/anthropics/skills --skill frontend-design -y
npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-composition-patterns -y
npx skills add https://github.com/shadcn/ui --skill shadcn -y
```

**Step 2 — scaffold.** Read `references/scaffold.md` and follow Path A (Vite + React + TypeScript). Don't ask about the stack; only switch to Next.js if the user explicitly requests it.

**Step 3 — write the project files.** Create `guidelines.md` and `design.md` in the project root (see "Project files" section below). guidelines.md gives the agent code conventions; design.md gives it the design system. Together they keep every future feature production-ready and visually consistent.

After start, the designer has a production-ready skeleton, the right skills installed, and both files their agent will follow as they build.

```
/vibe-to-prod start
```

## Scaffold mode (greenfield)

For designers starting a new project from scratch who don't need the full skill-install + guidelines setup. Sets up the production-ready architecture before any UI is built. Always Vite + React + TypeScript.

Read `references/scaffold.md` and follow it.

```
/vibe-to-prod scaffold
```

## Quick mode

Structure + types + stubs only. Dimensions: 1, 2, 3, 4, 7, 17, 18.

```
/vibe-to-prod quick [file or directory]
```

## Scope inference

If the user targets a specific concern ("just fix the state management"), apply only relevant dimensions. State which ones at the start.

---

# Target File Layout

```
src/
├── data/               # Extracted mock/seed data (never imported directly by components)
├── domain.ts           # Canonical data types (TypeScript interfaces)
├── schemas/            # Zod/Yup/Valibot runtime validation schemas
├── api/
│   ├── index.ts        # Async stubs with @backend annotations
│   └── hooks/          # TanStack Query / SWR hooks wrapping API stubs
├── components/
│   ├── ui/             # Shared reusable primitives (one per pattern)
│   ├── icons/          # Custom SVG icon components (only if no library equivalent)
│   └── ConfirmDialog.tsx
├── contexts/           # One isolated context provider per domain
├── hooks/              # Shared custom hooks (non-data-fetching)
├── routes/             # react-router setup + guards
├── pages/              # Route-level page components
└── utils/              # Helpers, formatters
.env.example            # Required environment variables (stubbed)
guidelines.md           # Code conventions the agent follows on every future feature
design.md               # Design system source of truth (colors, typography, spacing — designer's language)
```

---

# Project files: guidelines.md and design.md

## guidelines.md — code conventions

In **every mode** — start, scaffold, audit, refactor, quick — write a `guidelines.md` to the project root. This is the prevention layer: with it in place, the designer's everyday agent (Copilot, Cursor) follows the same production conventions while building new features — so the codebase stays clean and a future audit passes most dimensions without a heavy refactor.

**If `guidelines.md` already exists in the project:** read it first. Merge — don't blindly overwrite. Keep any project-specific rules the team already wrote; add the vibe-to-prod conventions they're missing. If it conflicts (e.g. it says "use PropTypes"), update it to match the TypeScript-always stance and tell the user what changed.

**If it doesn't exist:** create it covering the same ground as the dimensions, in build-time language: smart/dumb split, no hardcoded data, canonical TS types, API stubs with `@backend` markers, no fetch in components, design tokens, no AI slop, component reuse priority, resilience states, security, accessibility, the feature build sequence. (The full template lives in the skill's `guidelines.md` reference — use it as the basis.)

## design.md — design system source of truth

Also in every mode, ensure a `design.md` exists at the project root. This is the designer's document — written in design language, not code. It's structured so an agent can build new on-brand pages without seeing existing code.

A complete design.md has these sections (the full fillable template is in `references/design-md-template.md` — use it as the basis):

1. **Overview** — the brand feel, positioning, key characteristics
2. **Colors** — grouped by role (brand, surface, text, semantic), each with `{token.name}`, hex, and a usage note
3. **Typography** — families (display/body/mono), hierarchy table, principles
4. **Layout** — spacing scale, grid, whitespace philosophy
5. **Elevation & Depth** — elevation levels and when each applies
6. **Shapes** — border-radius scale, illustration style
7. **Components** — each component with token references, dimensions, variants (the most valuable section)
8. **Do's and Don'ts** — brand rules
9. **Responsive behavior** — breakpoints, collapsing strategy
10. **Known gaps** — what's not extractable, logo assets vs tokens, out-of-scope

The `{token.name}` syntax throughout is the bridge: the agent reads `{colors.primary}` in design.md and generates `--primary: #cc785c` in the theme CSS.

**design.md is the source of truth. The theme CSS tokens are generated from it.** When a designer updates design.md (adding a color, changing spacing), the agent regenerates the theme to match. The flow is always: design.md → theme CSS → components reference tokens. Never the reverse.

**If `design.md` already exists:** read it, respect it, use it to verify the current theme tokens match. Flag any drift (theme has colors not in design.md, or design.md defines tokens not yet in the theme). Merge in any missing standard sections.

**If it doesn't exist:** extract the design system from the existing codebase (scan for colors, fonts, spacing, component patterns) and write design.md in the full structure above. Then verify the theme matches. (For HTML conversion projects, the extraction is detailed in `references/html-to-react.md` step 2.)

**Designers can paste an external design.md.** Grabbing a design.md from another website or project is increasingly common. If a designer provides one, the agent reads it and regenerates the theme tokens to match — applying the new design system to the existing components.

## Agent instruction

After writing both files, instruct the user to add this to their agent's instructions (Copilot instructions, Cursor rules, or the agent's system prompt):

> "Read guidelines.md and design.md before building any feature. Follow the code conventions in guidelines.md. Use the design tokens defined in design.md for all colors, typography, and spacing — never use raw values."

This closes the loop: the skill sets up the project once, and both files keep every future feature production-ready and visually consistent without re-running the skill.
