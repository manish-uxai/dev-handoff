# Vibecode Smells Reference

> Loaded on every invocation. Quick reference for common patterns that need fixing.

---

## Smell table

| Smell | Signal | Fix |
| :--- | :--- | :--- |
| **Reinvented UI primitive** | Custom dropdown, modal, tooltip, tabs built from scratch | Replace with shadcn/Radix equivalent, preserve styling |
| **Duplicated component variants** | Same button/card built differently in 3+ files | Consolidate into one shared primitive in `components/ui/` |
| **Copy-pasted layouts** | Same page section structure repeated across pages | Extract into a shared layout component |
| **Direct mock data in component** | `import { MOCK_DATA }` inside a UI file | Route through `api/index.ts` stub |
| **Component fetching directly** | `fetch()` or `axios.get()` inside a component | Route through `api/index.ts` stub — all-or-nothing rule |
| **No empty/error/loading states** | Component shows blank or crashes with no data | Add all three fallback states |
| **Hardcoded labels in component** | `placeholder="Enter name"` baked in, not a prop | Pass as prop instead |
| **Hardcoded visual values** | Raw hex, `px` in inline styles, Tailwind `[17px]` | Use design tokens or standard Tailwind scale |
| **One file doing too many things** | 400+ line component mixing data fetching + UI | Split into container + presentational children |
| **useState chain (5+)** | 5+ independent hooks in one component | Consolidate into reducer or Zustand store |
| **Conditional routing** | `if (page === 'home')` in root component | Use react-router-dom or Next.js file routing |
| **Component crashes on null** | `patient.address.city` without null guard | Add `?.` optional chaining or null check |
| **Fixed widths breaking layout** | `width: 800px` on a component that should flex | Use relative sizing or max-width constraints |
| **Raw SVG in JSX** | 20+ line `<path d="M 12 3..."/>` block | Check lucide-react first, then heroicons, then extract |
| **Relative imports** | `../../components/Button` | Replace with `@/components/Button` |
| **setTimeout driving visuals** | `setTimeout(() => setVisible(true), 300)` | Use CSS transition or Framer Motion instead |
| **Missing @backend annotation** | API function with no `// @backend` comment | Add method, path, auth, and response shape |
| **No data-testid on interactive elements** | Buttons, forms, major sections unlabeled | Add `data-testid` attributes |

---

## Severity defaults

These are starting points — always justify the severity in context.

| Smell | Default severity |
| :--- | :--- |
| Component fetching directly | High |
| No @backend annotations | High |
| Conditional routing | High |
| No data-testid coverage | High |
| Missing loading/error/empty states | High |
| Component crashes on null | High |
| Direct mock data in component | High |
| One file doing too many things | Medium |
| Duplicated component variants | Medium |
| useState chain (5+) | Medium |
| Hardcoded visual values | Medium |
| Reinvented UI primitive | Medium |
| No PropTypes / incomplete types | Medium |
| Fixed widths | Medium |
| Raw SVG in JSX | Low |
| Relative imports | Low |
| Hardcoded labels | Low |
| Console.log in production | Low |
