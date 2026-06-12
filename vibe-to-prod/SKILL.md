---
name: vibe-to-prod
description: Transforms high-fidelity designer-built React or Next.js prototypes into production-ready codebases. Detects the project stack and adapts automatically. The output is clean, production-survivable code — not a documentation package. Use this skill when a designer wants to make their vibe-coded UI production-ready, when someone says "make this dev-ready", "clean up my prototype", "prep for integration", "vibe to prod", or any mention of turning a coded prototype into something a developer can ship.
license: MIT
compatibility: Works with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, and other agentskills.io-compatible agents. Supports React and Next.js projects. JavaScript codebases are migrated to TypeScript automatically — output is always TypeScript. Other stacks trigger guided redirection.
metadata:
  author: vibe-to-prod
  version: "6.0.0"
  framework: 18-dimension-handoff
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

## Dependency install (right after Node check passes)

Once Node is confirmed, check whether dependencies are installed. If `node_modules` is missing OR there's no lockfile (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`), run `npm install` automatically before proceeding. The designer should never have to do this manually — the skill handles it.

Two things this solves: the designer never has to find and use the terminal, and `npm audit` (dimension 17) needs a lockfile to work — fresh Figma Make exports ship without one, so installing first makes the security scan meaningful.

**Handle the common Figma Make install failure:** these exports sometimes contain a pnpm-style `link:` dependency in package.json that `npm install` can't resolve (fails with a workspace/link error). If install fails for this reason, fix the malformed dependency entry in package.json, then re-run install. Don't stop the whole run over it — patch and proceed.

If `npm install` fails for any other reason, report the exact error plainly and stop, rather than continuing on a broken dependency tree.

---

# Step 0: Stack Detection

**Identify what the user has built.** Check `package.json`, file extensions, and folder structure.

---

## React (with TypeScript) — `.tsx` / `.ts`

Proceed with all 18 dimensions.

---

## React (with JavaScript) — `.jsx` / `.js`

**vibe-to-prod always outputs TypeScript.** A JavaScript codebase is migrated to TypeScript as the mandatory first step — this is not optional and the user is not asked. TypeScript is the production-handoff standard: it gives the receiving developer real type contracts, autocomplete, and refactoring safety that JavaScript can't.

Tell the user plainly, then proceed:

> "Your project is in JavaScript. I'll convert it to TypeScript first — this gives your developer proper type safety and makes the handoff cleaner. Then I'll run the full production pass on the TypeScript version."

Read and follow `references/jsx-to-tsx-migration.md` to migrate, THEN run the 18 dimensions on the resulting TypeScript codebase. Because migration happens first, everything downstream is TypeScript — there is no JavaScript path through the dimensions, and PropTypes are never used (TypeScript interfaces replace them entirely).

---

## Next.js — `next.config.*` present or `next` in `package.json`

Proceed with all 18 dimensions, applying the Next.js overrides listed later in this document. If the Next.js project is in JavaScript, migrate to TypeScript first, same as above.

---

## Plain HTML / CSS / JS — no `package.json`, or only `.html` / `.css` files

The project isn't React yet, but the UI work has value. vibe-to-prod can convert it to a production-ready Vite + React + TypeScript project — the HTML becomes React components, inline styles become design tokens, and the result goes through the full 18-dimension pass.

Tell the designer:

> "Your project is built with plain HTML and CSS. I'll convert it to a React + TypeScript project first — your visual design stays the same, but it gets proper components, data flow, and production structure. Then I'll run the full production pass so a developer can start integrating immediately."

**Conversion flow:**

1. Scaffold a Vite + React + TypeScript project (Path A from `references/scaffold.md`)
2. Convert HTML to React — read and follow `references/html-to-react.md`
3. Run the 18 dimensions on the resulting React codebase

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

# The 18-Dimension Framework

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

### 3. Canonical Domain Types (`domain.ts`)

A single file that describes every data entity the UI renders. The codebase is TypeScript (migrated first if it arrived as JS), so this is always strict interfaces — never JSDoc. This is the dimension that prevents the "multiple discussion rounds" problem: when the data shape changes, a developer changes one file, not a parent component plus three children.

Strict interfaces in `domain.ts`:

```ts
export interface Patient {
  id: string;
  name: string;
  status: "active" | "inactive";
}
```

Components import their types from `domain.ts` — they don't define their own. One central source of truth for every shared entity (workspace, dashboard, study, etc.).

**Quality check:** No duplicated or conflicting interface definitions across the codebase. Every interface must be complete — all fields the UI actually uses must be typed. In audit mode, verify interfaces match actual runtime usage. Two completeness tells: (1) `as any` casts to reach fields not in the interface — the interface is missing those fields; (2) baked-in UI text (labels, placeholders, button copy hardcoded in a component rather than typed props) — reduces reusability, flag as Low. A developer should understand a component's API from its prop interface without reading its internals.

**Error handling, not runtime-validation libraries:** Do NOT add Zod, Yup, or any runtime-validation library. The developer integrating the real backend decides how responses are validated. What the handoff needs is plain try/catch around the async stub calls (dimension 4), with a visible error state when something fails — not a schema layer the backend dev didn't ask for and may not use. Keep it simple: typed interfaces for shape, try/catch for failure.

### 4. API Contract Stubs (`api.ts`) + Data Layer

This is the integration seam — the single place a developer swaps mock data for real API calls. Get this right and the dev plugs in a backend without touching the UI.

- **Hard rule:** No component imports mock data directly. All data flows through async functions in `api.ts`.
- Each function is an **async stub** that simulates latency and returns mock data:

```ts
// src/api.ts
import { MOCK_PATIENTS } from "./data/patients";
import type { Patient } from "./domain";

const delay = (ms: number) => new Promise((r) => setTimeout(r, ms));

// Returns the patient list. Swap the body for a real API call at integration.
export async function getPatients(): Promise<Patient[]> {
  await delay(300);
  return MOCK_PATIENTS;
}
```

- **Self-documenting names are the contract.** `getPatients()`, `getStudies()`, `getDocuments()` — a developer reading `api.ts` sees every data dependency and what it returns, from the function names and return types alone. No annotation system needed; the typed stub IS the map.
- **The function is the swap-point.** At integration, the developer replaces the stub body (`await delay; return MOCK`) with a real fetch/axios call. The UI calling `getPatients()` never changes.

**Do NOT add a data-fetching library.** No TanStack Query, no SWR, no `QueryClientProvider`. The developer wires up real data fetching their own way at integration time — that's explicitly their job, not the prototype's. Adding TanStack now creates a provider that must be mounted (a runtime-crash risk) and a pattern the dev may not use.

**Wire data through Context API.** For data that's shared across screens, expose it via a React Context provider that calls the `api.ts` stubs, simulating async with the `delay` above. This gives the dev a clean global data layer they extend at integration — and it's the pattern they actually expect (Context or Redux), not query hooks.

```tsx
// Components consume context, which calls the api.ts stub internally
const { patients, loading, error } = usePatientData();
```

- **Third-party integrations** (maps, charts, heatmaps) never fetch internally — data comes through the `api.ts` stubs like everything else.

## State & Access

### 5. State Management

- Avoid chains of 5+ independent `useState` hooks — a 30-40 useState component is the canonical mess.
- **Lead with a reducer.** For genuinely complex screens, consolidate scattered `useState` into a `useReducer` with explicit named actions (`SET_FIELD`, `ADD_ARM`, `TOGGLE_COUNTRY`). Named actions make state transitions visible and are the cleanest swap-point for the developer's eventual Context/Redux integration.
- **Simpler fallback — combine into one object.** Where a full reducer is overkill, collapse many related `useState` into a single state object updated together. Reduces the hook count and keeps related fields in one place.
- **Nested/repeating fields → `useFieldArray`** (or equivalent), so edits return the whole array of objects as one structured value rather than a tangle of per-field state.
- **Provider-mount safety:** when extracting state into a Context provider, you MUST mount that provider in the routing/app tree so it wraps every component that consumes its hook. Creating the provider and hook files is not enough — an unmounted provider makes the consuming hook throw at runtime, killing the screen, while build and tests still pass. After any provider change, trace: does this provider wrap every component calling its hook? Verify in the browser, not just the build.

### 6. Read-Only / Role-Based Access Control (conditional)

**Only applies if the app actually has roles or read-only states.** Many prototypes have no concept of permissions — if so, skip this dimension entirely and don't flag its absence as a problem. Check first: does the app distinguish viewer/editor/admin, or have any read-only mode? If not, this dimension is N/A.

If the app does have roles:

- Use the HTML `inert` attribute to block UI interaction for read-only roles (one declaration on a container beats per-element `disabled` props, which silently miss newly-added elements).
- Define roles as a union type in `domain.ts`.

### 7. Self-Documenting Data Layer

The developer must be able to open `api.ts` and see every data dependency they need to build a backend for — without a separate annotation system.

- Stub function names describe what they return: `getStudies()`, `getPatientDocuments()`, `getStudyById(id)` — not `fetchData()` or `loadStuff()`.
- Return types reference `domain.ts` interfaces, so the shape is visible from the signature.
- Each stub's body clearly returns mock data via the `delay` simulation, so the swap-point is obvious.
- Together this means a developer reads `api.ts` top to bottom and has the full list of endpoints to build and the exact shape each returns — the typed, well-named stub IS the integration map. No `@backend` comments, no separate contract file.

## UI Quality & Performance

### 8. Component Library Compliance & Reuse

The core problem: vibecoding produces duplicate components for the same thing (three different dropdowns, two custom modals) and hand-rolled primitives that a standard library does better and a developer would have to maintain. Don't hand off bespoke UI when a standard exists.

The rule, in priority order:

1. **Reuse what's already in the codebase.** If a component for this purpose already exists, use it — never create a second one that does the same thing. Duplicate components for the same job is the #1 issue to fix here.
2. **If none exists, import from shadcn/ui (or Radix).** Don't hand-roll a dropdown, modal, tooltip, tabs, popover, date picker, or select menu — these are solved primitives.
3. **Preserve genuine domain components.** A custom data grid or domain-specific visualization is legitimate; don't replace those.

- **Actively scan** for reinvented primitives and for duplicate components serving the same purpose across folders.
- Replace reinvented primitives with shadcn/Radix equivalents, preserving visual styling.
- Consolidate duplicates into one shared component.
- **Do not auto-pass.** If no scan was performed, mark as unevaluated.

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

### 10. The Missing Cases — Destructive, Empty, Error States + Real-Data Resilience

Vibecoding builds the happy path. The designer demos the flow that works and never hits the cases where data is missing, an action is destructive, or something fails — so those cases never get built. This dimension catches what vibecoding structurally omits. A complete UI handles them; a prototype usually doesn't.

**The missing cases:**

- **Destructive actions:** gate all deletions, resets, or irreversible changes behind `ConfirmDialog` or "Toast with Undo." Vibecoded prototypes almost always have a delete button with no confirmation.
- **Empty states:** every list, table, or data view needs a "no data yet" state — not a blank area or a crash on an empty array.
- **Error states:** every action that can fail needs a visible failure path — not a silent dead-end.

(Loading states are covered in dimension 14. Together, 10 and 14 cover the four cases vibecoding forgets: destructive, empty, error, loading.)

**Real-data resilience (the same omission, applied to data shape):** vibecoding renders the 3 mock items perfectly and never the messy real data. Check that components survive:

- **Null/undefined fields:** renders gracefully when a data field is missing, not crashes (this overlaps dimension 3 — missing fields are usually a type-completeness problem; flag under whichever is clearer)
- **Text overflow:** long names, descriptions, URLs wrap or truncate, not break the layout
- **Variable lengths:** works with 0, 1, and 500 items — not just the 3 mock items
- **Fixed widths:** no hardcoded pixel widths that prevent adapting (if responsive is explicitly out of scope, note and move on)

**Long-running asynchronous operations:** prototypes fake long operations with a frozen screen or a spinner that never resolves. Real agentic flows (LLM calls, document ingestion) can take many minutes — a user will not sit on a blocked screen. The correct pattern: move the user off the blocking screen (e.g. back to a dashboard), show a "processing" status for the operation, and give them a way to check back (a reload/refresh affordance) rather than holding the UI hostage. Flag any multi-second simulated operation that blocks the whole screen with no escape.

**Coverage method (scan all, deep-read capped):** run the null-access and fixed-width greps across ALL components, then deep-read the highest-risk flagged candidates capped at 8. Report how many were flagged vs deep-read — never "passed" on a small sample.

### 11. Routing & Navigation (conditional)

**First check: is this a single-page or multi-page app?** A single-screen prototype doesn't need routing — skip this dimension, don't flag its absence.

If multi-page:

- Is it already using a router? If yes, leave it (don't force a switch to a different router — that's the dev's choice).
- If it's switching views with conditional rendering (`if (page === 'home')`) and no router, convert to declarative `react-router-dom` paths. **Read and follow `references/conditional-to-router.md`** — it's a step-by-step recipe for this exact conversion with known pitfalls documented. Do NOT improvise the routing migration; the reference file exists because improvising it causes expensive retries.
- Lazy-load heavy route containers (`React.lazy`).

### 12. Expensive Operations (narrow)

**Do NOT add `useMemo`/`useCallback`/`React.memo` as a blanket practice.** React's own guidance discourages manual memoization — it's an anti-pattern when applied everywhere, and React 19's compiler handles it automatically. Missing memoization is NOT a finding. A developer profiles and adds it where measured.

Only flag the narrow real case: a genuinely expensive operation (sorting/filtering thousands of rows, heavy computation) that runs on every render or keystroke AND will visibly lag with real data volumes. This is rare in a prototype. If you don't see a concrete performance problem, this dimension passes — don't manufacture memoization work.

## Robustness & Production Readiness

### 13. Code Quality (top-tier — non-negotiable)

The codebase is TypeScript (migrated first if it arrived as JS). Code quality is held to a high bar — a developer should open any file and find it clean:

- **Zero `any` types.** Strict, complete interfaces. `any` defeats the entire point of TypeScript.
- **No `// @ts-ignore` or `as any` escape hatches** — each one hides a real bug.
- Organized, deduplicated imports. Absolute path aliases (`@/*`), never `../../` chains.
- No dead code, no unused variables, no commented-out blocks left behind.
- Consistent patterns — same approach to the same problem across files.
- ESLint/Prettier clean.
- No PropTypes — ever. TypeScript interfaces are the type contracts; PropTypes are redundant and must not be added. If migrating from JS, PropTypes are removed and replaced by interfaces, not kept alongside them.

### 14. Error Boundaries & Component Resilience

**Route-level:**

- Wrap major views in React Error Boundaries.
- Global Toast/Snackbar for async failures.

**Component-level (every data-consuming component must handle):**

- **Loading state:** what the user sees while data loads (skeleton, spinner, or placeholder — not blank space)
- **Error state:** what happens when the data call fails (fallback message — not a crash)
- **Empty state:** what appears when there's no data (helpful message — not invisible component)

### 15. Dependency, Environment & Onboarding Hygiene

- Clean unused packages. Pin critical dependency versions.
- Provide `.env.example` with all required environment variables stubbed.
- **README setup check:** A developer cloning this repo should be able to run the app within 5 minutes. Verify:
  - `README.md` exists and is not the default template
  - It includes install instructions (`npm install`)
  - It includes run instructions (`npm run dev`)
  - It mentions any required environment variables
  - If README is missing or only a default template, flag as Medium severity

### 16. File Hygiene & Icon Consolidation

- Delete orphaned files, unused imports, dead code.
- **Icon replacement (lookup first, extract last):**
  1. Find all inline SVGs in JSX components
  2. For each SVG, identify what it represents — use file name, component name, and surrounding code as context if the path is unclear
  3. Check **lucide-react** first (1500+ icons, most prototype icons have a match)
  4. If not found, check **heroicons** or **phosphor-icons**
  5. If a library equivalent exists — replace the entire SVG block with a single import line
  6. Only extract to a custom icon file in `components/icons/` if no reasonable library equivalent exists
  7. Never leave raw multi-line SVG coordinate paths inside UI components

### 17. Security Basics

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

### 18. Design Quality (No AI Slop)

This is the dimension designers care about most — it protects visual craft, not just code structure. Vibe-coded UIs drift toward generic "AI dashboard" tropes and inconsistent styling.

**What "AI slop" actually means here — the undifferentiated-default cluster:**

AI slop in UI has a specific signature. None of these is bad alone — they're bad _together_, because together they signal "generic default, no design decision was made":

- Inter or Geist as the default sans-serif
- Blue or indigo as the primary accent with subtle hover states
- Large rounded corners on every card and button
- A gradient hero section with headline, subheadline, and CTA
- Icon + label sidebar nav that looks straight out of default shadcn
- 50px+ padding everywhere because "clean and spacious" is the safe default

The test is NOT "does it use shadcn" — the skill mandates shadcn, and shadcn is good. The test is: **does this UI have an identity, or is it the undifferentiated default?** A project with a real `design.md` driving distinct brand colors, intentional type, and considered spacing uses shadcn AND looks like itself. Slop is shadcn shipped raw with default Inter, default blue, default radii, and no design.md identity applied.

So the check is: does `design.md` define a real identity, and do the components reflect it — or do they fall back to framework defaults? If the design.md exists but components ignore it and use defaults, flag that drift. If there's no identity at all (the default cluster above, all present together), flag it as slop. A single default (just using Inter, say) is fine — flag the _cluster_.

**Also avoid:**

- Decorative accent borders on cards (thick `border-left`/`border-right` to imply importance) — generic AI-dashboard cliché
- Random or one-off colors not defined as semantic tokens — every color should map to a token in `:root`
- Inline `style={{ color: '#…' }}` or ad-hoc hex/rgb in components
- Turning every data series the same alert color (e.g. all bars red when "critical") — this destroys metric identity

**Prefer:**

- The design.md identity applied consistently — brand colors, type, spacing that are intentional, not default
- Semantic tokens for everything: `--color-category-*`, `--color-status-*`, `--color-fg-muted`
- Legends that match what's drawn — color encodes the metric type, severity is a separate visual cue
- shadcn `Tooltip`, `Badge`, `Card` styled via tokens, without custom chrome borders

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

### 14. Error Boundaries & Resilience (Next.js)

Check for built-in error files instead of manual Error Boundaries:

- `error.tsx` at appropriate layout levels
- `not-found.tsx` for 404 handling
- `loading.tsx` for suspense states
  Component-level resilience checks (loading/error/empty states) still apply.

### 4 & 7. API Stubs & Self-Documenting Data Layer (Next.js)

The `api.ts` stub pattern still applies. Stub names/comments should note whether an endpoint is expected as a Next.js API route (`/app/api/resource/route.ts`) or an external service.

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

0. **Create a task list first, then work through it.** Before anything else, create a tracked todo list of the audit phases and tick each off as you complete it. This is the audit's completion anchor — the runs that tracked tasks didn't skip steps; the runs that dove in without a plan are the ones that forgot to write design.md or shortchanged the orphan pass. The phases: (1) pre-flight (node, npm install, build, npm audit), (2) reachability script for orphans, (3) grep evidence pack across dimensions, (4) tsc/depcheck tooling, (5) deep-read highest-risk files, (6) write design.md + guidelines.md, (7) compile report. Tick each only when actually done.

1. **Read before judging.** Before running any grep patterns, spend 2 minutes mapping the project: what is it for, what stack, what conventions are already in place? This prevents false positives — you won't flag JSDoc as a problem if you first understand it's a JS codebase by design.

2. **Lead with stack detection.** Identify the stack (React/TS, React/JS, Next.js, or unsupported). If unsupported, stop and follow Step 0 redirection. If Next.js, apply overrides. **If the codebase is JSX/JS, note at the top of the report that it will be migrated to TypeScript as the first refactor step** — don't audit it as a permanent JavaScript project, and don't flag "missing PropTypes" as a finding (PropTypes won't exist post-migration; TS interfaces replace them).

3. **Prefer 15 high-confidence findings over 50 speculative ones.** If you can't point to a specific file and line to prove a finding, don't include it. Speculative findings waste the reader's time and undermine trust in the real findings.

4. **Label facts vs judgments.** A fact is: "this function has no error handling: api.js:142." A judgment is: "this module's responsibilities feel unclear." Both are valid — but label which is which so the reader knows what's proven and what's an opinion.

5. **Show evidence, not conclusions.** Every failing dimension must cite specific file paths WITH line numbers.

   **Citation format rules (non-negotiable):**
   - Every citation must include a line number: `api.js:42` not just `api.js`
   - Citing the same file multiple times is ONLY acceptable if each citation has a DIFFERENT line number and describes a DIFFERENT issue
   - If you catch yourself writing `api.js, api.js, api.js` — STOP. That is padding. Write `api.ts:42 (getPatients returns untyped data), api.ts:67 (createPatient not a stub — direct mutation), api.ts:91 (component imports MOCK directly, bypasses api.ts)` instead.
   - If you cannot provide a line number, write "file:? (line not verified)" — this is honest and acceptable. Writing the same filename three times is not.

   **WRONG:** `Evidence: domain.js, domain.js, domain.js`
   **WRONG:** `Evidence: SoaWorkspace.jsx, SoaWorkspace.jsx`
   **RIGHT:** `Evidence: domain.js:45 (duplicate PopulationView typedef), domain.js:112 (conflicting Scenario fields)`
   **RIGHT:** `Evidence: SoaWorkspace.jsx:134 (delete handler), SoaWorkspace.jsx:201 (reset handler)`

   **Verify numbers before stating them.** If you report a file has 23,053 lines or a context has 26 useState calls — that number must come from an actual grep/awk output you ran, not from estimation. If you didn't run the count command, don't state a specific number. Say "very large file" or "many useState calls" instead. A wrong number destroys the credibility of the entire report.

6. **Run the grep patterns from audit-checklist.md.** Required, not optional. Include output summaries in the report.

6b. **Run the real tools, don't just grep proxies for them.** The strongest audits ran tools; the weakest hand-grepped. Make the tool the instruction so every model gets the strong result. Run these and report results (run-and-report — if one errors or hangs, note it and move on, never let it block the audit):

- **`npm run build`** — catches what static analysis misses: missing-file imports, broken exports. A failing build is a High finding (the dev can't even run it). The v5.6.0 test caught a build broken by missing data files this way.
- **`tsc --noEmit`** (once a tsconfig exists; if none exists, that absence is itself the dimension-14 finding) — the real type-safety signal. Report the error count instead of just grepping for `: any`.
- **`npx depcheck`** — finds unused dependencies and duplicate libraries (e.g. MUI icons shipped alongside lucide). Turns "I think this is unused" into fact for dimension 15.
- **`npm audit --audit-level=high`** — report the vulnerability count for dimension 17.

6c. **Provider-mount / runtime-crash check.** Build passing does NOT mean the app runs — a context provider can be missing while the build stays green. For every context hook the app calls (`useX()` from a `createContext`/`XProvider` pair), verify the matching provider is actually mounted above it in the App/main tree. The classic silent break: a `useAppData()` hook called in a component whose `AppDataProvider` was never mounted — every consumer throws "must be used within AppDataProvider" at runtime while the build passes. (This codebase uses Context API for shared data, not TanStack — so the providers to check are the app's own context providers.) Flag any hook-without-mounted-provider as High. This is the single highest-value runtime catch in the audit.

7. **Flag interactions from code.** Scan for extreme spring configs, zero-duration transitions, undebounced hover handlers, setTimeout-driven visuals. List findings with file + line, or explicitly state "No interaction smells found." Never skip this section.

8. **Dimension 4 is all-or-nothing.** Any component fetching directly = fail.

9. **Dimension 8 requires active scanning.** Search for reinvented primitives using the grep patterns in audit-checklist.md. If no scan was performed, mark as unevaluated — not passing.

10. **Run the reachability script ONCE for orphan detection — do not grep by hand.** Designers vibecode many screen variants; Figma Make exports generate a file for every screen. Orphaned variants are normal — but hand-grepping finds them unreliably (the same codebase returned 8, 24, and 57 orphans across test runs depending on which files the agent happened to check). **Run the script in `references/reachability.md`** — it walks the real import graph from the entry point (including `export...from`, dynamic `import()`, and `React.lazy`) and returns the complete, identical orphan list every time.

    The script splits results into two buckets that MUST be treated differently:
    - **App/variant orphans** (outside `ui/`): unused screen variants and Figma artifacts. List these in the dedicated "Orphaned files (skip for refactoring)" report section; safe to flag for deletion.
    - **ui/ library primitives** (e.g. `carousel.tsx`, `calendar.tsx`): unused shadcn/Radix components that are a _library_, not dead exploration. Note separately and neutrally. NEVER flag for deletion or refactoring — the designer may use them next week.

    The "Orphaned files" section must be complete — the fix-it-all reads it to know what to skip, which is what prevents wasted credits splitting dead god-components. If the script fails to run, fall back to per-file grep but state in the report that the orphan list may be incomplete.

11. **Real-data resilience coverage (dimension 10).** Run null-access and fixed-width greps across ALL components. Deep-read the highest-risk flagged candidates, capped at 8. Report how many were flagged vs deep-read — never report "passed" on a small sample.

12. **Dimension 17 requires security scanning.** Run the security grep patterns. Flag any hardcoded secret as High immediately. Run `npm audit --audit-level=high` and report the vulnerability count.

13. **Dimension 18 (design quality) — explain fully, never compress.** This is the designer's home turf. Flag AI-slop patterns, color-token drift, and severity-vs-metric color issues in full clear prose. These are usually Medium/Low but matter most to the audience.

14. **Every finding must include a severity (High/Medium/Low) with justification.** Not just the label — one sentence explaining why.

15. **Required output structure — always a table first, then details.** Every audit follows the same shape so the designer gets a consistent, scannable result every time (no more text-list one run, table the next):

    First, a **summary table** covering all 18 dimensions at a glance:

    | #   | Dimension              | Status | Severity |
    | --- | ---------------------- | ------ | -------- |
    | 1   | Component Architecture | FAIL   | High     |
    | 2   | Clean Data Extraction  | PASS   | —        |
    | ... | ...                    | ...    | ...      |

    Status is PASS / PARTIAL / FAIL / N/A, and the distinction matters — some models collapse everything to FAIL, which makes the report useless (a designer reading "everything failed" panics or stops trusting it). Use the grades precisely:
    - **PASS** — the dimension is present and sound. Not perfect, but a developer wouldn't object. (e.g. shadcn used throughout, no hand-rolled primitives.)
    - **PARTIAL** — present but incomplete. The right pattern exists somewhere but isn't applied consistently, or covers some cases not all. (e.g. domain.ts exists but only types 2 of 8 entities; some destructive actions confirm, others don't.)
    - **FAIL** — absent or broken. The thing genuinely isn't there, or exists but doesn't work. (e.g. no api.ts at all; build fails; XSS hole.)
    - **N/A** — conditional dimension that doesn't apply (RBAC on an app with no roles; routing on a single-page app).

    A codebase that already had routing/TS/reuse applied should NOT come back straight-FAIL — that's miscalibration. If you're marking nearly everything FAIL, re-check whether some are really PARTIAL.

    **Each dimension's finding must be its own.** Do not populate a dimension with another dimension's evidence — e.g. don't paste the XSS (security) finding into the Error Boundaries row to give it something to say. If a dimension has no genuine finding, mark it PASS or N/A. A borrowed finding is worse than an honest PASS.

    Then, **below the table, the detailed findings** — only for dimensions that aren't a clean PASS, each with evidence (file:line), the fact/judgment label, severity justification, and the plain-language consequence.

    After the findings, include a dedicated **"Orphaned files (skip for refactoring)"** section listing every file the reachability pass (rule 10) found unimported, with a one-line note that these are unused variants the fix-it-all will skip. This section must be complete — it's what the fix-it-all consults to avoid wasting credits on dead code. If none, say "None — all component files are reachable."

    This structure is required in both audit mode and the report a fix-it-all run produces at the end.

16. **Trust grep output — don't over-read files.** The grep evidence pack proves most findings on its own. Only deep-read a source file when the finding genuinely requires seeing code structure (dimension 10 null-handling, confirming a god-component's internal organization). Reading files to "double-check" a finding grep already proved wastes tokens. Cap verification reads to what's strictly necessary.

17. **Design fidelity note:** In audit mode, visual fidelity cannot be verified without running the app. State this once at the top: "Visual fidelity should be verified in the browser after any refactoring." Do not mark it as "AT RISK."

18. **End with a "Summary for the designer."** A warm, plain-language section (no jargon — use the translation table) covering:
    - Overall health in a sentence
    - Top 3 things to fix and why they matter for handoff
    - Roughly how much developer time the fixes will save
    - **Design system status:** is design.md present? Are the tokens consistent? Any color/spacing drift? This is the part designers care about most — tell them whether their design system is being respected or drifting
    - **Component library status:** are shadcn components being used, or are there hand-rolled alternatives?
    - **Visual fidelity note:** remind them to verify in the browser after any refactoring

    This is the part a non-technical reader actually reads. It should make the designer feel informed and in control.

19. **Self-review gate before finalizing.** Re-read your own report and verify each of these before sending. If any fail, fix before finalizing:
    - Every citation has a line number (or honest `file:? (line not verified)`)
    - No filename appears multiple times without distinct line numbers and distinct issues
    - Every specific number (line counts, hook counts) came from actual command output, not estimation
    - Every finding pairs a technical term with a plain consequence
    - Security and design-system findings are in full clear prose, not compressed
    - The summary table covers all 18 dimensions; detailed findings appear below it
    - The reachability pass ran and the "Orphaned files" section is present and complete (every flagged god-component was checked for importers — a large file with zero importers must be listed as orphaned, not as a god-component to split)
    - The designer summary contains no untranslated jargon

20. **End with a two-path choice.** After the designer summary, offer exactly two ways forward — no menu of individual tasks:

    > **How would you like to proceed?**
    >
    > 1. **Fix it all** — say "fix it all" or "let's go" and I'll fix every finding in one continuous pass, then report back when it's done.
    > 2. **Pick specific findings first** — tell me which to start with (e.g. "just the security and data issues") and I'll handle those, then check in about the rest.

    Then stop and wait. The designer's reply routes into refactor mode (see the two paths there).

```
/vibe-to-prod audit [file or directory]
```

## Refactor mode (default)

Full 18-dimension pass. Refactor while preserving design intent. The self-documenting stubs in `api.ts` (well-named typed async functions) are the integration map — no separate contract document is generated.

**Pre-flight:** Run `node --version` first. If Node.js is missing, stop and direct the designer to install it (see pre-flight section at top of this file).

**If the codebase is plain HTML (no React):** don't refuse — convert it. Scaffold a Vite + React + TypeScript project first (Path A from `references/scaffold.md`), convert the HTML files into React components (see Step 0 HTML detection for the conversion flow), then continue with the 18 dimensions on the resulting React codebase.

**Step zero — TypeScript migration is a HARD GATE (if the codebase is JSX/JS).** This is not a plan item that can be reordered or deferred — it is a blocking precondition. If the codebase is JSX/JS (React but not TypeScript), you MUST complete the full TypeScript migration BEFORE touching any other dimension. Do not fix the data layer, API envelopes, state, bundling, or anything else first. Nothing else happens until migration is done.

Read and follow `references/jsx-to-tsx-migration.md` completely:

- Install TypeScript toolchain (typescript, @types/react, @types/react-dom), create tsconfig.json
- Rename every `.jsx` → `.tsx` and `.js` → `.ts`
- Add interfaces for every component's props
- Remove all PropTypes and the `prop-types` dependency — interfaces replace them
- Convert domain.js JSDoc typedefs to domain.ts interfaces
- Run `tsc --noEmit` and resolve type errors (no `any` shortcuts)
- Confirm `test` and `build` pass on the migrated TypeScript codebase

**This is the single most-skipped step.** Past runs have done "easier wins" (API, bundling) first and silently never circled back to migration, leaving the codebase in JavaScript despite the skill promising TypeScript output. Do NOT do this. Migration is step zero — verify it's complete (zero `.jsx` files remain, tsconfig.json exists, prop-types removed) before proceeding to any dimension. If you find yourself fixing other things while `.jsx` files still exist, STOP and migrate first.

### Two paths — how the designer drives the work

**Path 1 — "Fix it all" (the designer said "fix it all," "fix everything," "let's go," "make it production ready").**

This is explicit authorization to fix everything — even if it takes two rounds. The designer chose hands-off; they do NOT want to make decisions or project-manage. Your job is to complete all the work, not to hand them a list.

**Two categories of work, handled differently:**

**Category A — deterministic hardening (do ALL of it in one continuous pass, no stops):** TypeScript migration (if needed), data extraction, domain types, API stubs (Context API + setTimeout, no TanStack), state consolidation + provider-mount safety, RBAC (if applicable), self-documenting data layer, component library/reuse, design tokens, the missing cases (destructive/empty/error), routing migration, expensive-ops check, code quality, error boundaries, dependency/env hygiene, file hygiene/icons, production readiness, security, design quality. Every one of these is bounded and safe. Work through them continuously — no "want me to continue?", no task menus, no "what remains" lists.

**Category B — high-risk surgery (the ONE thing that gets its own round):** decomposing god-components (splitting 1,000–2,000 line files into smaller pieces). This is the single riskiest change — most likely to introduce a runtime break, and most dangerous when attempted late in a long run as context degrades. It gets handled as a deliberate, separately-authorized second round. (Note: only the _file-splitting surgery_ is Category B. The rest of dimension 1 — DOM hierarchy, circular deps, aliases, reuse — is Category A and runs in round one.)

**The flow:**

1. **Write a progress ledger first.** Before any edits, create `vibe-to-prod-progress.md` at the project root. Top of file, verbatim: "CONTINUOUS FIX-IT-ALL PASS. Do not present a menu. Do not stop on safe work. Resume the next unchecked item automatically." Then list every finding as a checkbox in priority order, tagged [A] or [B]. This is the anti-compaction anchor — on a large codebase the conversation WILL compact, and after compaction you re-read this file to recover both the plan and the no-menu rule. Tick a box ONLY after that item's build/runtime check passes (not when you believe it's done).

2. **Get the orphan list before any edits — run the reachability script.** Designers vibecode many screens and variants; a Figma Make export of 175 files can have 10-20 orphaned variants nobody uses. You MUST identify all of them up front, reliably.

   - If the audit already produced an "Orphaned files" section, start from that list.
   - Otherwise (or to confirm), run the script in `references/reachability.md` — it walks the real import graph and returns the complete app/variant orphan list plus the separate ui/ primitive list. Do not grep by hand; it misses files.
   - Mark every app/variant orphan as `SKIP — orphaned` in the ledger.

   **Do not edit, refactor, split, fix types in, replace icons in, or do ANY work on an orphaned file** — not in round one, not in round two, regardless of how many findings it has or how large it is. A 2,000-line orphaned god-component is skipped entirely; an orphaned file with an `any` type is skipped entirely. Every edit to a dead file is wasted credits. (The unused ui/ primitives are also left alone — they're a library, not dead code.)

   This is not optional. Build the full orphan list once, before the first edit, and consult it before touching any file. In prior runs, missing this wasted ~20% of the run splitting and editing files that were never imported.

3. **Announce the plan once, then proceed:**

   > "Got it — fixing everything. I'll work through the data layer, types, security, the missing states, routing, and cleanup in one continuous pass, verifying as I go. The one thing I'll hold for a separate step is splitting your largest files — that's the riskiest change and safer done deliberately. Starting now."

4. **Work continuously through all Category A items** in priority order (High → Medium → Low). Narrate progress so the designer can follow without responding: "✓ Data layer and API stubs done. Now adding destructive-action confirmations..." No stops, no menus.

5. **If the run pauses for length/compaction:** re-read the ledger and resume automatically. Say "Continuing — next is X" and keep going. NEVER present options to resume.

6. **When all Category A is done and verified, hand off to round two with a reflexive prompt.** This is the ONE legitimate stop. It is NOT a menu (not a list, not "if you want, next I'll…"). It's a single clearly-bounded item with a reason, phrased so the designer types "go" reflexively — because they already said fix it all, so continuing is the expected default, not a new decision:

   > "Everything's hardened and verified — data layer, types, security, missing states, routing, all done and building clean. One thing left: your [N] largest _in-use_ files need splitting into smaller pieces. That's the riskiest change, so I held it for its own focused pass where I verify each split in the browser as I go. **Say "go" and I'll finish it.**"

7. **On "go" (round two):** split the god-components one at a time, verifying each in the browser/runtime before the next. **Only split files that passed the reachability scan** — orphaned god-components stay untouched regardless of their size. This is the highest-breakage-risk work — go carefully, never batch it. When done, report and remind the designer to click through their screens.

**Why round two exists:** not because the agent can't do the work, and not as a menu in disguise — but because splitting huge files is where runtime breaks hide, and doing it as its own verified pass (rather than hastily at the tail of a long, context-degraded run) is what protects the designer's app. The "say go" prompt makes continuing effortless — one reflexive word, not a decision.

**Never, in Path 1, produce any of these (they are all the forbidden menu):**

- A numbered list of remaining tasks
- "What remains: …"
- "If you want, next I'll…"
- "I can continue with X, Y, or Z"
- Any turn that ends with safe Category-A work outstanding and a request to proceed

The only permitted stops are: (a) a genuine blocker (build won't compile, real design ambiguity), or (b) the single round-two handoff for god-component splitting, phrased as above.

**Clean up the ledger.** `vibe-to-prod-progress.md` is a working file, not a deliverable. Add it to `.gitignore` when you create it, and delete it once the whole fix-it-all (both rounds) is complete — it shouldn't ship in the handoff.

**Non-blocking warnings are NEVER work.** A "large bundle chunk" / chunk-size warning is not a finding, not a dimension, not a functional break. Do NOT optimize Vite chunks. Do NOT list it under "what remains." Do NOT offer it as a follow-up. Do NOT mention it as outstanding. If the build succeeds with a chunk warning, that dimension of work is DONE. (This has been ignored in past runs — the agent chased chunk optimization across multiple passes. Stop. It is out of scope, full stop.)

**Path 2 — "Pick specific findings" (the designer named specific things: "just the security and data issues").**

1. Fix the named findings in one pass (same continuous, no-menu behavior — just scoped to what they named).
2. When done, offer the next step as a single yes/no with a one-line reason in designer language:
   > "Done with the security and data fixes. Want me to also tackle the state management next? It's what keeps the app fast and easy for a developer to extend. Type yes to continue — or tell me you're done and I'll leave the rest for handoff."
3. If yes → fix the next finding(s), then offer the next again. Repeat until all findings are done.
4. **Always offer a clean exit.** The designer can stop partway and hand off what's fixed. "Tell me you're done" is always a valid response — don't pressure them to fix everything.

### Validation — build-pass is NOT done-done

**Critical: passing tests and a successful build do NOT mean the app works.** A missing provider, a dead route, or a broken screen will still pass `npm run build` because they're runtime errors, not compile errors. Tests pass, build succeeds, app is broken. This has happened — a refactor added a context provider but never mounted it in the tree, killing an entire module, while tests and build stayed green.

So validation has three layers, in order:

1. **Type check + build:** `tsc --noEmit` (no type errors), `npm run build` (compiles). Necessary, not sufficient.
2. **Tests:** `npm run test`. Necessary, not sufficient.
3. **Runtime verification — the real check:** the build passing only proves it compiles. Confirm the app actually renders. Where you can, start the dev server and check that the main routes render without throwing. At minimum, reason through every screen a refactor touched and confirm it will still render.

**Provider-mount check (a specific, predictable failure):** Any time you add or move a React context provider, you MUST verify the provider actually wraps the components that consume its hook, in the real routing/app tree — not just that the provider and hook files exist. The classic break: add `XProvider` and `useX()`, forget to mount `XProvider` around the route that calls `useX()`, and the hook throws at runtime. After adding any provider, trace: does this provider wrap every component that calls its hook? If not, the app is broken even though it builds.

**Batch validation cadence:** group related edits and run type-check + test + build at meaningful checkpoints (after finishing a dimension or logical group), not after every few files. But after any change involving providers, routing, or lazy-loaded modules, do the runtime/provider check immediately — these are the changes that break silently.

### Completion reporting — never claim "done" on build alone

When reporting completion (Path 1 final report, or Path 2 between steps), NEVER say "complete, all verified" based only on test+build. Always end with a runtime-verification ask in plain designer language:

> "Tests and build pass. Before you hand off, please open the app in your browser and click through your main screens — the dashboard, the population view, any forms — to confirm everything renders. Some issues only show up at runtime, not in the build. Tell me if any screen looks wrong and I'll fix it."

For a designer who can't read code, that browser click-through is the real test. Make it a required step, not a footnote.

### Working principles (both paths)

**Never edit an orphaned file.** Before doing ANY work on a file — splitting, type fixes, icon replacement, anything — check whether that file is actually imported by another file in the app. Designers vibecode many screen variants; Figma Make exports generate files for every variant in the Figma file. Orphaned variants are normal and expected. Editing them is wasted credits. `grep -r "import.*ComponentName" src/` takes seconds and can save hundreds of credits. This applies to both Category A and Category B work, in both Path 1 and Path 2.

**Avoid over-engineering. "Fix it all" means fix every finding well — not apply every possible change maximally.** Use judgment:

- Split god-components by concern (Category B, round two), but don't decompose files that are already cohesive
- Target fixed widths that genuinely break responsive layout; leave acceptable micro-constraints
- Fix data flow before routing — data boundary cleanup gives bigger handoff value
- Don't add memoization as a blanket practice (dimension 12 — it's a React anti-pattern); only where a measured problem exists
- The goal is a codebase a developer can integrate, not one that satisfies every linter rule at the cost of churn

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
├── domain.ts           # Canonical data types (TypeScript interfaces — one source of truth)
├── api/
│   └── index.ts        # Self-documenting async stub functions (setTimeout-simulated)
├── components/
│   ├── ui/             # Shared reusable primitives (one per pattern)
│   ├── icons/          # Custom SVG icon components (only if no library equivalent)
│   └── ConfirmDialog.tsx
├── contexts/           # One isolated context provider per domain (shared data + state)
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

**If it doesn't exist:** create it covering the same ground as the dimensions, in build-time language: smart/dumb split, no hardcoded data, canonical TS types, API stubs as self-documenting async functions, no fetch in components, design tokens, no AI slop, component reuse priority, resilience states, security, the feature build sequence. (The full template lives in the skill's `guidelines.md` reference — use it as the basis.)

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
