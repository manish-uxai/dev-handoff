# 16-Dimension Handoff Audit Checklist

Copy this checklist when auditing a vibecoded prototype for backend handoff.

```
Pre-check:
- [ ] Visual Fidelity — UI renders identically before and after changes

Handoff Audit Progress:
- [ ] 1. Component Architecture — logic/presentation split; break god-components
- [ ] 2. Data Extraction — MOCK_/seed data in /src/data/, not inline in UI
- [ ] 3. Canonical Domain Types — domain.ts as strict API contract
- [ ] 4. API Contract Stubs — typed async api.ts with simulated latency
- [ ] 5. State Management — context/reducers with loading/error/optimistic states
- [ ] 6. Read-Only / RBAC — guarded setters; inert for read-only roles
- [ ] 7. Backend Integration Markers — @backend annotations + BACKEND_CONTRACT.md
- [ ] 8. CSS Token Compliance — semantic CSS variables, no hardcoded hex
- [ ] 9. Destructive Action Safety — ConfirmDialog for delete/reset
- [ ] 10. Routing & Navigation — react-router-dom, lazy routes, route guards
- [ ] 11. Component Performance — useMemo, useCallback, React.memo on heavy lists
- [ ] 12. Accessibility — semantic HTML, ARIA, keyboard nav, modal focus trap
- [ ] 13. Linting, Formatting & Typing — no any, strict TS, clean imports, ESLint
- [ ] 14. Error Boundaries & Resilience — route error boundaries, toast on API errors
- [ ] 15. Dependency & Environment Hygiene — no unused deps, pinned versions, .env.example
- [ ] 16. File Hygiene — no orphans, unused imports, dead code, consistent naming
```

## Quick grep patterns

Run these to spot common violations:

```bash
# God components (files over 500 lines)
find src -name '*.tsx' -exec awk 'END{if(NR>500)print FILENAME, NR}' {} \;

# Inline mock data in components
rg "MOCK_|const \w+ = \[" src/components src/pages --glob '*.{tsx,ts,jsx,js}'

# Hardcoded colors
rg "#[0-9a-fA-F]{3,8}|rgb\(|hsl\(" src --glob '*.{tsx,css,scss}'

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

# useState chains (5+ in one component)
rg -l "useState" src --glob '*.tsx' | xargs -I {} sh -c 'count=$(rg -c "useState" {}); [ "$count" -ge 5 ] && echo "{}: $count useState calls"'
```

## Vibecode smell quick-check

| Smell | Command |
|-------|---------|
| God component | `find src -name '*.tsx' -exec awk 'END{if(NR>500)print FILENAME,NR}' {} \;` |
| Duplicated Tailwind | `rg -o 'className="[^"]{80,}"' src --glob '*.tsx' \| sort \| uniq -d` |
| No TypeScript usage | `rg -L "interface\|type " src --glob '*.tsx'` |
| Missing loading states | `rg -L "loading\|isLoading\|skeleton\|Spinner" src --glob '*.tsx'` |

## Target file layout (handoff-ready)

```
src/
├── data/               # Extracted mock/seed data
├── domain.ts           # Canonical business types (API contracts)
├── api.ts              # Typed async stubs with @backend annotations
├── components/
│   ├── ui/             # Presentational primitives
│   └── ConfirmDialog.tsx
├── contexts/           # One provider per domain
├── hooks/              # Shared custom hooks
├── routes/             # react-router setup + guards
├── pages/              # Route-level page components
└── utils/              # Helpers, formatters
.env.example            # Required env vars (stubbed)
BACKEND_CONTRACT.md     # Generated integration doc for dev team
```
