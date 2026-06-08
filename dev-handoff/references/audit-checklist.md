# 15-Dimension Handoff Audit Checklist

Copy this checklist when auditing a prototype for backend handoff.

```
Handoff Audit Progress:
- [ ] 1. Component Architecture — logic/presentation split; one provider per domain
- [ ] 2. Data Extraction — MOCK_/seed data in /src/data/, not inline in UI
- [ ] 3. Canonical Domain Types — domain.ts as strict API contract
- [ ] 4. API Contract Stubs — typed async api.ts with simulated latency
- [ ] 5. State Management — guarded slices; audit/undo/versioning where needed
- [ ] 6. Read-Only / RBAC — guarded setters; inert for read-only roles
- [ ] 7. CSS Token Compliance — semantic CSS variables, no hardcoded hex
- [ ] 8. Destructive Action Safety — ConfirmDialog for delete/reset
- [ ] 9. File Hygiene — no orphans, unused imports, dead code
- [ ] 10. AI Apply Mechanism — PARAMETER_UPDATERS lookup tables
- [ ] 11. Routing & Navigation — react-router-dom, lazy routes, route guards
- [ ] 12. Component Performance — useMemo, useCallback, React.memo on heavy lists
- [ ] 13. Accessibility — semantic HTML, ARIA, keyboard nav, modal focus trap
- [ ] 14. Linting, Formatting & Typing — no any, strict TS, clean imports, ESLint
- [ ] 15. Error Boundaries & Resilience — route error boundaries, global toast on API errors
```

## Quick grep patterns

Run these to spot common violations:

```bash
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
```

## Target file layout (handoff-ready)

```
src/
├── data/           # MOCK_ arrays, seed fixtures
├── domain.ts       # Canonical business types
├── api.ts          # Typed async backend stubs
├── components/
│   ├── ui/         # Presentational primitives
│   └── ConfirmDialog.tsx
├── contexts/       # One provider per domain
├── routes/         # react-router setup + guards
└── utils/
    └── PARAMETER_UPDATERS.ts
```
