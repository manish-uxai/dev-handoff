# Design & Development Guidelines

> **For the AI agent:** Read this file before building any feature and follow its conventions. Every component, type, and data flow you create should match these rules. This keeps the codebase production-ready as it grows.
>
> **Scope**: Conventions for building production-ready, dev-handoff-ready React features.
> Follow these while building so the codebase stays clean — and you rarely need a heavy audit later.
> **Stack**: Vite + React + TypeScript + shadcn/ui + Tailwind + semantic CSS tokens. (TypeScript always — JS exports get migrated to TS.)

These guidelines are the "build it right the first time" companion to the vibe-to-prod skill. If you follow them as you go, a final audit should pass most dimensions cleanly.

---

## 1. Smart / Dumb Component Split

Separate state ownership from rendering.

- Smart components (context providers, containers) own domain state, side effects, data fetching.
- Dumb components (cards, tabs, modals, screens) receive data via props/context and call setters. They never hold domain state.
- If a file has both state logic and 200+ lines of JSX, split it. Never let a file exceed 400 lines.

Why: swapping a UI component becomes a pure presentation change, and a dev can trace data flow in one file.

---

## 2. No Hardcoded Data in Components

All mock data, seed data, dropdown options, and static lists live in `/src/data/`. No inline arrays in component files.

Why: backend integration becomes "swap one function body," not "hunt through 30 files."

---

## 3. Canonical Domain Types

One source of truth for every data entity.

- Interfaces in `/src/types/domain.ts` (or `/src/domain.ts`) — this project is TypeScript
- No duplicated or conflicting definitions. Every type complete — all fields the UI uses.
- Add runtime validation (Zod, or whatever's already installed) so bad API data is caught at the boundary.

---

## 4. API Stubs With Backend Markers

Every data operation is a typed async function in `/src/services/api.ts` (or `/src/api/index.ts`).

- Returns mock data with simulated latency (`await delay(300)`) today; swap the body for a real call later.
- **Every stub carries a `// @backend` annotation** with method, path, auth, and response shape. This is the developer's integration map — don't skip it.
- Components consume data through these functions (or hooks wrapping them), never by importing `/src/data/` directly and never via inline `fetch()`.

```
// @backend GET /api/patients
// Auth: Bearer token (JWT)
// Response: { data: Patient[]; meta: { total: number } }
```

Wrap stubs in a data-fetching hook layer (TanStack Query or whatever's installed) so components get loading/error/caching states automatically.

---

## 5. Data Fetching — Never Fetch Inside Components

No component calls `fetch()` or `axios` directly — not even maps, charts, or third-party widgets. All external data flows through the API layer. One violation means a developer has to rewrite component logic to connect a real backend.

---

## 6. CSS Design Tokens — No Hardcoded Visual Values

Never use raw hex/rgb, or raw pixel values, in component files. Use semantic CSS tokens from your theme file.

| Prefix                               | Purpose              |
| ------------------------------------ | -------------------- |
| `--primary`, `--secondary`           | Core UI theme        |
| `--color-action-*`                   | Primary actions      |
| `--color-shell-*`                    | Shell-level controls |
| `--foreground`, `--muted-foreground` | Text                 |
| `--background`, `--card`, `--muted`  | Surfaces             |
| `--border`, `--input`, `--ring`      | Interactive elements |
| `--destructive`                      | Danger/error         |
| `--color-status-*`                   | Status indicators    |

This covers colors AND spacing/sizing. No `p-[17px]`, no `style={{ padding: '12px' }}`, no `margin: 16px` in CSS. Snap to the Tailwind scale or add a token.

```tsx
// Correct
<div className="bg-[var(--card)] border border-[var(--border)] text-[var(--foreground)]">
// Incorrect
<div className="bg-white border border-gray-200" style={{ color: '#717182' }}>
```

If a needed color doesn't exist as a token: add it to the theme file with a semantic name first. Never inline "just for now."

---

## 7. Visual Design — No AI Slop

UI must look intentional and on-brand, not like a generic AI dashboard.

**Never:**

- Decorative accent borders on cards (thick `border-left`/`border-right` to imply importance)
- Random one-off colors not defined as tokens
- Recoloring an entire chart series the same alert color (all bars red when "critical") — this destroys metric identity

**Always:**

- Semantic tokens for every color
- Legends that match what's drawn — color = metric type, severity = separate cue
- Tooltips on flagged elements: what the metric is, its value, and why it matters
- shadcn `Tooltip`, `Badge`, `Card` without custom chrome

**Severity vs metric color — keep two layers separate:**

- Fill color = which metric (patient / site / cost each get their own token)
- Ring / badge / icon = severity (warning, critical)

Never replace metric color with severity color. Severity is an overlay.

---

## 8. Component Reuse Priority

Before creating a new component, check what exists:

1. Existing project component — use it, don't duplicate
2. shadcn/ui primitives (`/src/components/ui/`) — buttons, dialogs, dropdowns, switches, badges, inputs, selects
3. AI Elements (`/src/components/ai-elements/`) — chat, canvas, agent UI
4. Existing tokens + Tailwind layout classes on top of primitives
5. New token + new component — last resort

Never hand-roll a button, dropdown, modal, or tooltip when shadcn has one. Never ship raw unstyled native controls.

---

## 9. State Management

- No chains of 5+ `useState` hooks in one component — consolidate into a reducer or store (Zustand, or whatever's installed).
- Read-only enforcement: guarded setters + the HTML `inert` attribute on editable containers. Define roles as a union type (TS) or constants object (JS).

---

## 10. Routing

Use real routing (`react-router-dom`, or file-based for Next.js). Never switch views with `if (page === 'home')` conditional rendering — it blocks deep-linking and route guards.

---

## 11. Resilience — Every Data Component Handles Three States

For every component that consumes async data:

- **Loading:** skeleton or spinner, never blank space
- **Error:** fallback message, never a crash
- **Empty:** helpful message ("No results"), never an invisible component

Wrap major views in an error boundary so one failure doesn't crash everything.

---

## 12. Production Readiness — Survive Real Data

- Guard against null/undefined fields (`data?.field`) — don't assume data is always complete
- Handle text overflow (truncate or wrap long strings) — don't let them break layout
- Work with 0, 1, and 500 items — not just the 3 mock items
- Pass labels and placeholders as props, don't bake them into components
- No fixed pixel widths that break responsive layout

---

## 13. Accessibility

- Semantic HTML5 (`<nav>`, `<main>`, `<header>`, `<button>`) — no div soup
- `aria-label` on interactive elements, focus trapping in modals, visible focus states
- `aria-live` regions for dynamic updates
- (shadcn/Radix primitives give you most of this automatically)

---

## 14. Code Quality

- TypeScript everywhere: no `any`, strict interfaces, organized imports. (This project is TypeScript — if you started from a JS/JSX export, it gets migrated to TS first. No PropTypes; interfaces are the contracts.)
- ESLint/Prettier clean — no unused variables, exhaustive `useEffect` deps
- Absolute path aliases (`@/*`), never `../../` relative imports

---

## 15. Security

- Never hardcode API keys, tokens, or secrets in source — use `.env`, and add `.env` to `.gitignore`
- Provide `.env.example` with stubbed variable names
- Avoid `dangerouslySetInnerHTML`; if unavoidable, sanitize with DOMPurify
- Run `npm audit` and resolve high/critical vulnerabilities

---

## 16. Hygiene & Handoff

- No orphaned files, unused imports, or `console.log` in production code
- Inline SVGs: use a library icon (lucide-react) first; only create a custom icon file if no equivalent exists
- README has install + run instructions so a developer can start in 5 minutes
- No circular imports

---

## Feature Build Sequence

When adding a feature, follow this order so it's born production-ready:

1. **Types** — add domain types to the canonical file
2. **Mock data** — create `/src/data/mock-[feature].ts`, barrel-export it
3. **API stubs** — add typed async functions with `// @backend` markers
4. **Component** — build it: consume data via API/hooks, use tokens, reuse shadcn primitives, handle loading/error/empty, no inline data or hardcoded colors
5. **Navigation** — wire into routing
6. **Docs** — update README if it adds a screen, data file, or API domain

If you build every feature this way, the vibe-to-prod audit will pass most dimensions on the first run — and handoff is fast and cheap.
