# 19-Dimension Audit Checklist

Copy this checklist when auditing a vibe-coded prototype for production readiness.

## Audit output format

When running `/vibe-to-prod audit`, output must strictly follow:

```markdown
## Handoff Audit: [Scope / Module Name]

### Stack Detected
- **Stack:** [React+TS / React+JS / Next.js / Unsupported]
- **Adaptations:** [Next.js overrides applied / none]

### Visual Fidelity (audit mode)
- **Note:** Visual fidelity should be verified in the browser after any refactoring. Not evaluated in this code-only audit.

### Flagged Interactions (code-level scan)
| File:Line | Pattern | Reason |
|-----------|---------|--------|
| `src/components/X.tsx:88` | `stiffness: 800` | Likely jank |

Or: `No interaction smells found.`

### Dimension 1 — Reusability (subsection)
| Pattern | Files | Suggested Shared Primitive |
|---------|-------|---------------------------|
| Duplicate button variants | `src/A.tsx`, `src/B.tsx` | `components/ui/Button` |

Or: `None found after full codebase scan.`

### Violations
| # | Dimension | Issue | Severity | Evidence |
|---|-----------|-------|----------|----------|
| 4 | API Contract Stubs | [plain language + why severity] | High | `path:line` — distinct per row |

### Dimension 19 — Production Readiness Spot-Check
| Component | Null/Empty | Long Text | Hardcoded Content | Verdict |
|-----------|------------|-----------|-------------------|---------|
| `PatientCard` | fails on null name | OK | label baked in | Medium |

### Vibecode Smells Detected
- **[Smell]:** [File:line + details]

### Grep Evidence Summary
- [Required: summarize grep output below]

### Recommended Fix Order
1. **[Dimension #] - [Fix in plain language]**: Why first.

### Dimensions Passed
- **Dimension [X] - [Name]**: Why (with evidence).

### Dimensions Unevaluated
- **Dimension [X] - [Name]**: Why scan was skipped.
```

## Severity scale

| Severity | Meaning |
| :--- | :--- |
| **High** | Will break in production, block API integration, or cause data loss. Fix before handoff. |
| **Medium** | Creates friction for the developer or degrades user experience. Fix soon after handoff. |
| **Low** | Cleanup that improves code quality. Can be addressed over time. |

Every violation row must include one sentence explaining why that severity was chosen.

## Plain language rule

Before writing any finding in Issue or Recommended Fix columns, rewrite using the translation table in SKILL.md if the user is non-technical. File paths and grep output stay in Evidence only.

## Checklist

```
Step 0:
- [ ] Stack detected — React+TS, React+JS, or Next.js (if unsupported, stop and redirect)

Visual fidelity note:
- [ ] State once at top: "Visual fidelity should be verified in the browser after any refactoring." Do NOT use "AT RISK"

Plain language:
- [ ] All Issue/Recommended Fix text rewritten for non-technical users

Flagged Interactions:
- [ ] List findings with distinct file:line per row, or "No interaction smells found" — never skip
- [ ] Same file cited twice only with different line numbers and separate explanations

Audit Progress:
- [ ] 1.  Component Architecture — god-components broken; stateful/presentational split; DOM hierarchy guard
- [ ] 1 — Reusability subsection — scan ENTIRE codebase: similar filenames, overlapping classNames, copy-pasted JSX, same-purpose files; report as subsection NOT separate dimension row
- [ ] 2.  Data Extraction — MOCK_/seed data in /src/data/, not inline in components
- [ ] 3.  Canonical Domain Types — domain.ts/js complete; typedefs match runtime usage; no duplicates
- [ ] 4.  API Contract Stubs — all data through api.ts/js; @backend annotations; NO direct fetch (ALL OR NOTHING)
- [ ] 5.  State Management — context/reducers; no 5+ useState chains
- [ ] 6.  Read-Only / RBAC — inert attribute; roles in domain file
- [ ] 7.  Backend Integration Markers — @backend annotations match domain file shapes (in code, not separate docs)
- [ ] 8.  Component Library Compliance — ACTIVELY SCAN reinvented primitives; UNEVALUATED if scan skipped
- [ ] 9.  Design Token Compliance — colors, spacing, sizing, typography; no raw px in inline styles, CSS, or Tailwind arbitrary values
- [ ] 10. Destructive Action Safety — ConfirmDialog or Toast+Undo
- [ ] 11. Routing & Navigation — react-router-dom or Next.js file-based routes
- [ ] 12. Component Performance — useMemo, useCallback, React.memo
- [ ] 13. Accessibility — semantic HTML, ARIA, keyboard nav, focus trapping, aria-live
- [ ] 14. Code Quality — TS: no any; JS: PropTypes + quality JSDoc on api/domain
- [ ] 15. Error Boundaries & Resilience — route boundaries + loading/error/empty on data components
- [ ] 16. QA Test Selectors — data-testid on primary interactive elements
- [ ] 17. Dependency & Environment Hygiene — no unused deps, .env.example
- [ ] 18. File Hygiene & Icons — no orphans; SVGs over 10 lines extracted
- [ ] 19. Component Production Readiness — spot-check 3-5 components: null/empty/long text/hardcoded content/prop clarity/style isolation
```

---

## Evidence rules

Every failing dimension must include specific **`path:line`** references or grep output. Not opinions.

- **Banned:** same file cited twice as two evidence rows without different line numbers and separate explanations.
- **Banned:** merging Dimension 1 Reusability findings into the Dimension 1 Architecture row — use the Reusability subsection instead.
- **Required:** run grep patterns below; summarize in Grep Evidence Summary.
- **Required:** severity + one-sentence justification on every violation row.

---

## Quick grep patterns

**Required in audit mode. Include output summaries in the report.**

```bash
# === STEP 0: STACK DETECTION ===
grep -E '"next"' package.json 2>/dev/null && echo "Next.js detected"
find src -name '*.tsx' -o -name '*.ts' 2>/dev/null | head -3
find src -name '*.jsx' -o -name '*.js' 2>/dev/null | head -3

# === DIMENSION 1: ARCHITECTURE ===
find src \( -name '*.tsx' -o -name '*.jsx' \) -exec awk 'END{if(NR>400)print FILENAME, NR}' {} +

# === DIMENSION 1 — REUSABILITY SUBSECTION ===
# Duplicate basenames across folders
find src \( -name '*.jsx' -o -name '*.tsx' \) -exec basename {} \; | sort | uniq -d
# Similar component filenames
find src \( -iname '*button*' -o -iname '*card*' -o -iname '*badge*' -o -iname '*input*' \) | sort
# Overlapping long className strings
grep -RhoE 'className="[^"]{60,}"' src --include='*.jsx' --include='*.tsx' 2>/dev/null | sort | uniq -d | head -20
# Repeated prop-shape patterns
grep -RE 'variant.*size|size.*variant' src/components --include='*.jsx' --include='*.tsx' -l 2>/dev/null | head -20

# === DIMENSION 2 & 4: DATA FLOW ===
grep -RInE 'MOCK_|from.*/data/' src/components src/pages --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' 2>/dev/null | head -40
grep -RInE 'const\s+\w+\s*=\s*\[' src/components src/pages --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40
grep -RInE 'fetch\(|axios\.|\.get\(|\.post\(' src/components src/pages --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' 2>/dev/null | head -40

# === DIMENSION 3: DOMAIN TYPES / JSDOC QUALITY ===
grep -n '@typedef' src/domain.js 2>/dev/null || echo "No domain.js typedefs"
grep -nE 'interface |type ' src/domain.ts 2>/dev/null || echo "No domain.ts interfaces"
grep -c '@property' src/domain.js 2>/dev/null || true

# === DIMENSION 7: BACKEND MARKERS ===
grep -nE 'export\s+(async\s+)?function' src/api.ts src/api.js src/services/api.ts src/services/api.js 2>/dev/null | grep -v '@backend'
grep -RInE 'localhost|127\.0\.0\.1|https?://' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' --exclude='*.test.*' 2>/dev/null | head -40

# === DIMENSION 8: COMPONENT LIBRARY ===
grep -RInEi 'dropdown|modal|tooltip|popover|datepicker|date-picker|tablist|select.*option' src/components --include='*.jsx' --include='*.tsx' 2>/dev/null | grep -Evi 'node_modules|shadcn|radix|headlessui|/ui/' | head -40

# === DIMENSION 9: DESIGN TOKENS (colors, spacing, sizing, typography) ===
grep -RInE '#[0-9a-fA-F]{3,8}|rgb\(|hsl\(' src --include='*.jsx' --include='*.tsx' --include='*.css' --include='*.scss' 2>/dev/null | head -40
grep -RInE '\[#[0-9a-fA-F]{3,8}\]' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40
grep -RInE '(m|p|gap|w|h|text|leading|tracking)-\[' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40
grep -RInE '\[[0-9.]+px\]|\[[0-9.]+rem\]|\[[0-9.]+em\]' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40
grep -RInE "style=\{\{[^}]*(padding|margin|width|height|fontSize|gap|lineHeight|letterSpacing|fontWeight)[^}]*[0-9]+px" src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40
grep -RInE '(padding|margin|width|height|font-size|gap|border-radius|line-height|letter-spacing):\s*[0-9]+px' src --include='*.css' --include='*.scss' 2>/dev/null | head -40

# === DIMENSION 10: DESTRUCTIVE ACTIONS ===
grep -RInE 'onClick=\{.*delete|\.remove\(|reset\(' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40

# === DIMENSION 5 & 11: STATE & ROUTING ===
grep -RInE 'page === |currentPage|setPage\(' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40
find src \( -name '*.jsx' -o -name '*.tsx' \) -exec sh -c 'c=$(grep -c useState "$1" 2>/dev/null); [ "$c" -ge 5 ] && echo "$1: $c useState calls"' _ {} \;
grep -RInE 'react-router-dom|BrowserRouter|Routes|Route|useNavigate' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -20

# === DIMENSION 14: CODE QUALITY ===
grep -RInE ':\s*any\b|as any' src --include='*.tsx' --include='*.ts' 2>/dev/null | head -40
echo "PropTypes files: $(grep -RIl 'PropTypes' src --include='*.jsx' --include='*.js' 2>/dev/null | wc -l | tr -d ' ') / $(find src -name '*.jsx' 2>/dev/null | wc -l | tr -d ' ') jsx components"

# === DIMENSION 15: RESILIENCE ===
grep -RInE 'ErrorBoundary|componentDidCatch|getDerivedStateFromError' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -20
grep -RInE 'isLoading|loading|Skeleton|Spinner|emptyState|empty-state|no data|no results' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40

# === DIMENSION 16: QA SELECTORS ===
find src/components \( -name '*.jsx' -o -name '*.tsx' \) -exec sh -c 'grep -q data-testid "$1" || echo "$1"' _ {} \; 2>/dev/null | head -40

# === DIMENSION 17 & 18: HYGIENE ===
npx depcheck --skip-missing 2>/dev/null
grep -RInE 'console\.(log|warn|error)' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' --exclude='*.test.*' 2>/dev/null | head -40
grep -RIl '<svg' src/components src/pages --include='*.jsx' --include='*.tsx' 2>/dev/null | head -20
test -f .env.example && echo ".env.example present" || echo ".env.example missing"

# === DIMENSION 19: PRODUCTION READINESS ===
grep -RInE 'props\.\w+\.' src/components --include='*.jsx' --include='*.tsx' 2>/dev/null | grep -v '?.' | grep -v '&&' | head -30
grep -RInE "placeholder=['\"][A-Z]|label=['\"][A-Z]" src/components --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40
grep -RInE 'width:\s*[0-9]{3,}px|w-\[[0-9]{3,}px\]|minWidth:\s*[0-9]{3,}' src/components --include='*.jsx' --include='*.tsx' 2>/dev/null | head -30

# === FLAGGED INTERACTIONS ===
grep -RInE 'stiffness:\s*[5-9][0-9]{2,}|damping:\s*[0-4]\b' src --include='*.jsx' --include='*.tsx' 2>/dev/null | head -30
grep -RInE 'transition.*0ms|transition.*0s[^0-9]|duration-0' src --include='*.jsx' --include='*.tsx' --include='*.css' 2>/dev/null | head -30
grep -RInE 'setTimeout|setInterval' src/components src/pages --include='*.jsx' --include='*.tsx' 2>/dev/null | head -40
grep -RInE 'onMouse|onHover|onFocus.*set[A-Z]' src/components --include='*.jsx' --include='*.tsx' 2>/dev/null | head -30
```

---

## Vibecode smell quick-check

| Smell | Grep |
|-------|------|
| God component (400+ lines) | `find src \( -name '*.tsx' -o -name '*.jsx' \) -exec awk 'END{if(NR>400)print FILENAME,NR}' {} +` |
| Duplicate component basenames | `find src \( -name '*.jsx' -o -name '*.tsx' \) -exec basename {} \; \| sort \| uniq -d` |
| Similar button/card files | `find src \( -iname '*button*' -o -iname '*card*' \)` |
| Direct mock import | `grep -RInE 'MOCK_\|from.*/data/' src/components src/pages` |
| Direct fetch in component | `grep -RInE 'fetch(\|axios\.' src/components src/pages` |
| Reinvented primitive | `grep -RInEi 'dropdown\|modal\|tooltip' src/components \| grep -Evi 'radix\|shadcn\|/ui/'` |
| Tailwind arbitrary spacing | `grep -RE '(m\|p\|gap\|w\|h)-\[' src --include='*.jsx'` |
| Inline style raw pixels | `grep -RE "style=\{\{[^}]*(padding\|fontSize\|width)[^}]*px" src --include='*.jsx'` |
| Weak JSDoc (JS) | `grep -c '@property' src/domain.js` vs entities used in UI |
| Missing loading/empty states | `grep -RIL 'loading\|Skeleton\|empty' src/components` |
| Missing @backend | `grep -nE 'export\s+(async\s+)?function' src/api.js \| grep -v '@backend'` |
| No null guards (dim 19) | `grep -RE 'props\.\w+\.' src/components \| grep -v '?.'` |
| Hardcoded labels (dim 19) | `grep -RE "placeholder=['\"][A-Z]" src/components` |

---

## Target file layout

**React + TypeScript (default):**

```
src/
├── data/               # Extracted mock/seed data (never imported directly by components)
├── domain.ts           # Canonical data types
├── api.ts              # Async stubs with @backend annotations
├── components/
│   ├── ui/             # Shared reusable primitives — one per pattern
│   ├── icons/          # Extracted SVG icon components
│   └── ConfirmDialog.tsx
├── contexts/           # One isolated context provider per domain
├── hooks/              # Shared custom hooks
├── routes/             # react-router setup + guards
├── pages/              # Route-level page components
└── utils/              # Helpers, formatters
.env.example            # Required env vars (stubbed)
```

**React + JavaScript:** same structure; use `domain.js` and `api.js`.

**Next.js:** use `app/` or `pages/` for routing; add `middleware.ts`, `error.tsx`, `loading.tsx`, and `not-found.tsx` where appropriate.
