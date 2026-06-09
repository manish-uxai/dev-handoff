# 18-Dimension Handoff Audit Checklist

Copy this checklist when auditing a vibecoded prototype for backend handoff.

```
Pre-checks:
- [ ] Design Surface Fidelity — layout, spacing, animations, interactions render identically before and after
- [ ] Implementation Improvements — any animation/interaction refactors preserve perceived feel, not just code shape
- [ ] Buggy interactions flagged — suspicious flickers, jank, or broken states noted for developer, not silently preserved

Handoff Audit Progress:
- [ ] 1.  Component Architecture — logic/presentation split; god-components broken; duplicated UI patterns consolidated into shared primitives in components/ui/
- [ ] 2.  Data Extraction — MOCK_/seed data in /src/data/, not inline in UI components
- [ ] 3.  Canonical Domain Types — domain.ts as strict TypeScript API contract
- [ ] 4.  API Contract Stubs — typed async api.ts with simulated latency; all stubs carry @backend annotations; no component imports mock data directly
- [ ] 5.  State Management — context/reducers with loading/error/optimistic states
- [ ] 6.  Read-Only / RBAC — guarded setters; inert attribute for read-only roles
- [ ] 7.  Backend Integration Markers — @backend annotations match domain.ts; BACKEND_CONTRACT.md generated
- [ ] 8.  Component Library Compliance — reinvented primitives replaced with shadcn/Radix; genuine custom components preserved; replaced components flagged in BACKEND_CONTRACT.md
- [ ] 9.  CSS Token Compliance — semantic CSS variables, no hardcoded hex codes remaining
- [ ] 10. Destructive Action Safety — ConfirmDialog or Toast+Undo gates all delete/reset actions
- [ ] 11. Routing & Navigation — react-router-dom, lazy routes, route guards
- [ ] 12. Component Performance — useMemo, useCallback, React.memo on heavy lists
- [ ] 13. Accessibility — semantic HTML, ARIA, keyboard nav, modal focus trap, aria-live regions
- [ ] 14. Linting, Formatting & Typing — no any, strict TS, clean imports, ESLint/Prettier
- [ ] 15. Error Boundaries & Resilience — route error boundaries, toast on API failures, loading skeletons
- [ ] 16. QA Test Selectors — data-testid on all primary interactive elements and layout sections
- [ ] 17. Dependency & Environment Hygiene — no unused deps, pinned versions, .env.example present
- [ ] 18. File Hygiene & Icons — no orphans, unused imports, dead code; inline SVGs over 10 lines extracted to icons/
```

---

## Quick grep patterns

Run these to spot common violations:

```bash
# God components (files over 400 lines)
find src -name '*.tsx' -exec awk 'END{if(NR>400)print FILENAME, NR}' {} \;

# Direct mock data imports inside components or pages (hard rule violation)
rg "MOCK_|from.*\/data\/" src/components src/pages --glob '*.{tsx,ts,jsx,js}'

# Inline mock arrays in components (not routed through api.ts)
rg "const \w+ = \[" src/components src/pages --glob '*.{tsx,ts,jsx,js}'

# Hardcoded colors (CSS token violation)
rg "#[0-9a-fA-F]{3,8}|rgb\(|hsl\(" src --glob '*.{tsx,css,scss}'

# Hardcoded Tailwind arbitrary values (candidates for CSS variables)
rg "\[#[0-9a-fA-F]{3,8}\]|\[[\d.]+px\]" src --glob '*.{tsx,jsx}'

# Conditional routing anti-pattern
rg "page === |currentPage|setPage\(" src --glob '*.{tsx,jsx}'

# any types
rg ":\s*any\b|as any" src --glob '*.{tsx,ts}'

# Destructive actions without confirm
rg "onClick=\{.*delete|\.remove\(|reset\(" src --glob '*.{tsx,jsx}'

# Hardcoded API URLs
rg "localhost|127\.0\.0\.1|http://|https://" src --glob '*.{tsx,ts,jsx,js}' --glob '!*.test.*'

# Missing @backend annotations in api.ts
rg "export (async )?function" src/api.ts | rg -v "@backend"

# Unused dependencies
npx depcheck --skip-missing 2>/dev/null

# Console.log in production code
rg "console\.(log|warn|error)" src --glob '*.{tsx,ts,jsx,js}' --glob '!*.test.*'

# useState chains (5+ in one component — reducer candidate)
rg -l "useState" src --glob '*.tsx' | xargs -I {} sh -c 'count=$(rg -c "useState" {}); [ "$count" -ge 5 ] && echo "{}: $count useState calls"'

# Inline SVG blocks over 10 lines (extract to icons/)
rg -l "<svg" src/components src/pages --glob '*.{tsx,jsx}'

# Reinvented UI primitives (candidates for shadcn/Radix replacement)
rg -i "dropdown|modal|tooltip|popover|datepicker|date-picker|tablist" src/components --glob '*.{tsx,jsx}' | rg -v "node_modules|shadcn|radix|headlessui"

# Duplicated component class names (reusability smell)
rg -o 'className="[^"]{60,}"' src --glob '*.tsx' | sort | uniq -d
```

---

## Vibecode smell quick-check

| Smell | Grep Command |
|-------|---------|
| God component (400+ lines) | `find src -name '*.tsx' -exec awk 'END{if(NR>400)print FILENAME,NR}' {} \;` |
| Direct mock import in component | `rg "MOCK_\|from.*\/data\/" src/components src/pages --glob '*.tsx'` |
| Reinvented primitive | `rg -i "dropdown\|modal\|tooltip\|popover" src/components --glob '*.tsx' \| rg -v "radix\|shadcn"` |
| Duplicated component variants | `rg -o 'className="[^"]{60,}"' src --glob '*.tsx' \| sort \| uniq -d` |
| No TypeScript interfaces | `rg -L "interface\|type " src --glob '*.tsx'` |
| Missing loading states | `rg -L "loading\|isLoading\|skeleton\|Spinner" src --glob '*.tsx'` |
| Hardcoded hex in Tailwind | `rg "\[#[0-9a-fA-F]{3,8}\]" src --glob '*.tsx'` |
| Inline SVGs (10+ lines) | `rg -l "<svg" src/components src/pages --glob '*.tsx'` |
| Missing @backend annotations | `rg "export (async )?function" src/api.ts \| rg -v "@backend"` |
| Missing data-testid | `rg -L "data-testid" src/components --glob '*.tsx'` |

---

## Target file layout (handoff-ready)

```
src/
├── data/               # Extracted mock/seed data (never imported directly by components — always via api.ts)
├── domain.ts           # Canonical business types (API contracts)
├── api.ts              # Typed async stubs with @backend annotations
├── components/
│   ├── ui/             # Shared reusable primitives — one per pattern (Button, Card, Badge, Input...)
│   ├── icons/          # Extracted SVG icon components (inline SVGs over 10 lines moved here)
│   └── ConfirmDialog.tsx
├── contexts/           # One isolated context provider per domain
├── hooks/              # Shared custom hooks
├── routes/             # react-router setup + guards
├── pages/              # Route-level page components
└── utils/              # Helpers, formatters
.env.example            # Required env vars (stubbed)
BACKEND_CONTRACT.md     # Generated integration doc — includes replaced primitives flagged for dev swap
```