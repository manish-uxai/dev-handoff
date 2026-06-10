---
name: vibe-to-prod
description: Transforms high-fidelity designer-built React or Next.js prototypes into production-ready codebases. Detects the project stack and adapts automatically. The output is clean, production-survivable code — not a documentation package. Use this skill when a designer wants to make their vibe-coded UI production-ready, when someone says "make this dev-ready", "clean up my prototype", "prep for integration", "vibe to prod", or any mention of turning a coded prototype into something a developer can ship.
license: MIT
compatibility: Works with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, and other agentskills.io-compatible agents. Supports React and Next.js projects (TypeScript or JavaScript). Other stacks trigger guided redirection.
metadata:
  author: vibe-to-prod
  version: "4.4.0"
  framework: 20-dimension-handoff
---

# Philosophy

Traditional handoff is broken. A designer spends weeks building a polished, high-fidelity prototype in code. Then they hand over a Figma file or screenshots. The frontend developer spends another 1–2 months rebuilding the exact same UI from scratch. Fidelity degrades. Micro-interactions get lost. Time is wasted twice.

**We break this cycle entirely.**

The designer-builder has already developed the final production UI. Your job is to harden the code so it survives real data, real users, and real edge cases — then a developer opens the project and starts integrating APIs immediately. **1–2 months of UI development saved.**

**The output is production-ready code, not a documentation package.** No separate contract files. No lists of what was changed. The code itself is the deliverable — clean enough that a developer can read it and understand everything they need.

---

# Step 0: Stack Detection

**Before anything else, identify what the user has built.** Check `package.json`, file extensions, and folder structure.

---

## React (with TypeScript) — `.tsx` / `.ts`
Proceed with all 20 dimensions.

---

## React (with JavaScript) — `.jsx` / `.js`
Fully supported. Proceed with all 20 dimensions — the skill automatically applies the JavaScript-appropriate approach for each one. No warnings, no recommendations unless the user asks.

**TypeScript migration offer:** If the developer has indicated they prefer TypeScript, offer migration before running the 19 dimensions:

> "Your project is in JavaScript and that works fine. Your developer mentioned they prefer TypeScript — want me to migrate the codebase first? I'll do it properly: full interfaces in `domain.ts`, typed API stubs, PropTypes converted to interfaces, no shortcuts. Once that's done I'll run the full handoff audit on the TypeScript version."

If the user confirms, read and follow `references/jsx-to-tsx-migration.md` before proceeding with the 20 dimensions.

---

## Next.js — `next.config.*` present or `next` in `package.json`
Proceed with all 20 dimensions, applying the Next.js overrides listed later in this document.

---

## Plain HTML / CSS / JS — no `package.json`, or only `.html` / `.css` files
Do not proceed. Explain:

> "Your project is built with plain HTML and CSS. This skill is designed for React-based projects — the component structure, data flow patterns, and production-readiness checks are all React-specific.
>
> If we convert your prototype to React first, everything gets structured and ready — components, data flow, API connections — and a developer can skip straight to integration. That could save them 4-6 weeks.
>
> Want me to help convert your HTML prototype to React?"

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

- Good: "PropTypes missing on 143 components — a developer has to open each file to learn what data it expects."
- Good: "Direct fetch in PopulationMapView.jsx:42 — bypasses the API layer, so swapping in a real backend later means rewriting the component."
- Avoid: "PropTypes missing." (term with no teaching consequence)
- Avoid: "Components don't say what they need." (consequence with no term — designer learns nothing)

**Exceptions — always full, clear prose (never compressed):**
- Security warnings (hardcoded keys, XSS risks)
- Irreversible or destructive actions
- Anything design-system or design-related (this is the technical area designers engage with most — explain it fully)

**Summary: warm and plain.**
The end-of-audit summary is the one section written purely for a non-technical reader. No jargon. Covers overall health, the top 3 things to fix, and what it means for handing off to a developer. This is where the translation table applies.

| Technical term (in summary, translate to) | Plain version |
| :--- | :--- |
| god-component | one file doing too many things |
| API stubs / data layer | where real data will plug in |
| design tokens | reusable color and spacing values |
| error/empty states | what users see when data fails or is missing |
| PropTypes / types | components describing what data they expect |
| routing | how the app moves between pages |
| hardcoded secret | a password or key exposed in the code |

Read the user's messages for cues on how technical to make the summary. If they use "component", "props", "state" correctly, the summary can carry a bit more vocabulary.

---

## Severity scale

Every finding must include a severity level with this definition:

| Severity | Meaning |
| :--- | :--- |
| **High** | Will break in production, block API integration, or cause data loss. Fix before handoff. |
| **Medium** | Creates friction for the developer or degrades user experience. Fix soon after handoff. |
| **Low** | Cleanup that improves code quality. Can be addressed over time. |

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

Components must never import mock data arrays directly or call `fetch()` internally. All data must flow through async functions in `api.ts` or `api.js`.

**This applies to every component without exception** — maps fetching GeoJSON, charts loading CSV/JSON, tables pulling datasets. If a component gets data from anywhere other than a prop or an API stub function, it violates this rule.

When the developer connects real APIs, they change only the function body inside `api.ts`/`api.js`. Every component updates automatically.

---

# The 20-Dimension Framework

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

### 3. Canonical Domain Types (`domain.ts` / `domain.js`) + Runtime Validation

A single file that describes every data entity the UI renders.

**If TypeScript:** strict interfaces in `domain.ts`.
```ts
export interface Patient {
  id: string;
  name: string;
  status: 'active' | 'inactive';
}
```

**If JavaScript:** JSDoc typedefs in `domain.js`.
```js
/**
 * @typedef {Object} Patient
 * @property {string} id
 * @property {string} name
 * @property {'active' | 'inactive'} status
 */
```

**Quality check (both paths):** No duplicated or conflicting type definitions across the codebase. Every typedef must be complete — all fields the UI actually uses must be documented. In audit mode, verify existing typedefs match actual runtime usage.

**Runtime validation (catches bad API responses before they crash the UI):**

Check `package.json` for an existing validation library first:
- If **Zod** is present — add Zod schemas in `src/schemas/` alongside domain types
- If **Yup** is present — declare schemas using `yup.object()` and extract types with `yup.InferType<typeof Schema>`
- If **Valibot** or **Joi** is present — use that library's equivalent schema pattern
- If **nothing** is present — add Zod as the default: `npm install zod`

The principle is the same regardless of library: every API response shape should be validated at runtime so bad data from a real API is caught at the boundary, not deep inside a component.

```ts
// Example with Zod
import { z } from 'zod';

export const PatientSchema = z.object({
  id: z.string(),
  name: z.string(),
  status: z.enum(['active', 'inactive']),
});

export type Patient = z.infer<typeof PatientSchema>;
```

### 4. API Contract Stubs (`api.ts` / `api.js`) + Data Fetching Layer

- **Hard rule:** No component imports mock data directly. All data flows through async functions in `api.ts` or `api.js`. This rule is non-negotiable regardless of the data fetching library in use.
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
import { useQuery } from '@tanstack/react-query';
import { getPatients } from '../api';

export function usePatients() {
  return useQuery({
    queryKey: ['patients'],
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

**JavaScript annotation format:**
```js
// @backend GET /api/patients
// Auth: Bearer token (JWT)
// Response: { data: Patient[], meta: { total: number } }
export async function getPatients() {
  await delay(300);
  return { data: MOCK_PATIENTS, meta: { total: MOCK_PATIENTS.length } };
}
```

## State & Access

### 5. State Management & Reducer Isolation

- Avoid chains of 5+ independent `useState` hooks.
- Consolidate into reducer-based state with explicit action schemas. Allows developers to hook up Redux or Zustand instantly.

### 6. Read-Only / Role-Based Access Control

- Use the HTML `inert` attribute to block UI interaction for read-only roles.
- **TypeScript:** define roles as union type in `domain.ts`.
- **JavaScript:** define roles as constants object in `domain.js`.

### 7. Backend Integration Markers

- Every API stub annotated with `// @backend` as shown in dimension 4.
- Annotations must match data shapes in `domain.ts`/`domain.js`.
- The annotations live in the code — a developer reading `api.ts`/`api.js` can see every endpoint they need to build.

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

**If TypeScript:**
- Eradicate `any` types. Strict interfaces.
- Organized imports. ESLint/Prettier compliance.

**If JavaScript:**
- PropTypes on all components.
- JSDoc on all functions in `api.js` and `domain.js`.
- Organized imports. ESLint/Prettier compliance.

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
- **Baked-in UI text:** labels, placeholders, button text, error messages baked directly into a component instead of passed as props. (Note: this is UI *strings* — distinct from dimension 2, which covers hardcoded *data arrays*.) Baked-in text reduces reusability — flag as Low.
- **Prop interface clarity:** PropTypes (JS) or TypeScript interfaces (TS) define what the component accepts. A developer should understand the component's API without reading its internals.
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
The `api.ts`/`api.js` stub pattern still applies. `// @backend` annotations should note whether endpoints are expected as Next.js API routes (`/app/api/resource/route.ts`) or external services.

---

# Common Vibecode Smells

| Smell | Fix |
| :--- | :--- |
| **Reinvented UI primitives** | Replace with shadcn/Radix, preserve styling |
| **Duplicated component variants** | Consolidate into one shared primitive |
| **Copy-pasted layouts** | Extract repeated page sections into shared components |
| **Direct mock data in components** | Route all data through `api.ts`/`api.js` stubs |
| **Third-party components fetching directly** | Maps, charts fetching their own data — route through API stubs |
| **No empty/error/loading states** | Component shows blank or crashes with no data — add fallbacks |
| **Hardcoded content in components** | Labels, placeholders baked in — pass as props instead |
| **Hardcoded visual values** | Raw pixels, hex colors in inline styles or arbitrary Tailwind — use design tokens |
| **God-components (400+ lines)** | Split into container + children, guard the DOM hierarchy |
| **Cascading useState chains** | 5+ hooks — consolidate into reducer or context |
| **Components crash on null data** | Add defensive checks for missing fields |
| **Fixed widths breaking layout** | Use relative sizing or constrained max-widths |
| **Raw multi-line SVGs in JSX** | Check lucide-react first, then heroicons/phosphor. Replace with library import if match found. Only extract to custom icon file if no match exists. |
| **Hardcoded API key in source** | Move to `.env` and add `.env` to `.gitignore`. Never commit real keys. |
| **dangerouslySetInnerHTML** | Replace with safe rendering or sanitize input with DOMPurify. |
| **No tests exist** | A dev inherits no safety net. At minimum configure a test runner and add tests to critical paths. |
| **README missing setup instructions** | A dev can't run the app without asking the designer. Add install + run steps. |
| **Circular imports** | Components importing each other in a loop. Breaks builds and causes subtle bugs. |

---

# Execution Modes

## Audit mode

Listen for `/vibe-to-prod audit`. Report violations without modifying files.

Output format must follow [references/audit-checklist.md](references/audit-checklist.md).

**Audit rules:**

1. **Read before judging.** Before running any grep patterns, spend 2 minutes mapping the project: what is it for, what stack, what conventions are already in place? This prevents false positives — you won't flag JSDoc as a problem if you first understand it's a JS codebase by design.

2. **Lead with stack detection.** Identify the stack (React/TS, React/JS, Next.js, or unsupported). If unsupported, stop and follow Step 0 redirection. If Next.js, apply overrides.

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

12. **Every finding must include a severity (High/Medium/Low) with justification.** Not just the label — one sentence explaining why.

13. **Trust grep output — don't over-read files.** The grep evidence pack proves most findings on its own. Only deep-read a source file when the finding genuinely requires seeing code structure (dimension 19 null-handling, confirming a god-component's internal organization). Reading files to "double-check" a finding grep already proved wastes tokens. Cap verification reads to what's strictly necessary.

14. **Design fidelity note:** In audit mode, visual fidelity cannot be verified without running the app. State this once at the top: "Visual fidelity should be verified in the browser after any refactoring." Do not mark it as "AT RISK."

15. **End with a "Summary for the designer."** A warm, plain-language section (no jargon — use the translation table) covering: overall health in a sentence, the top 3 things to fix and why they matter for handoff, and roughly how much developer time the fixes will save. This is the part a non-technical reader actually reads.

16. **Self-review gate before finalizing.** Re-read your own report and verify each of these before sending. If any fail, fix before finalizing:
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

Full 20-dimension pass. Refactor while preserving design intent. The `// @backend` annotations in `api.ts`/`api.js` are the only integration contract — no separate document is generated.

**Avoid over-engineering. When the user says "fix it all," that does not mean apply every possible change maximally. Use judgment:**
- Split god-components, but start with data-bound ones — don't decompose every large file in one pass; some are fine as-is
- Don't globally remove every fixed pixel width — target ones that genuinely break responsive layout, leave acceptable micro-constraints
- Don't migrate routing before fixing data flow — data boundary cleanup gives bigger handoff value first
- Don't add memoization everywhere — only where it measurably helps
- The goal is a codebase a developer can integrate, not a codebase that satisfies every linter rule at the cost of churn

```
/vibe-to-prod refactor [file or directory]
```

## Scaffold mode (greenfield)

For designers starting a new project from scratch. Sets up the full production-ready architecture before any UI is built.

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
├── domain.ts           # Canonical data types (or domain.js with JSDoc)
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
```
