# 18-Dimension Handoff Audit Checklist

Copy this checklist when auditing a vibecoded prototype for backend handoff.

## Audit output format

When running `/vibe-to-prod audit`, output must strictly follow:

```markdown
## Handoff Audit: [Scope / Module Name]

### Stack Detected
- **Stack:** [React+TS / React+JS / Next.js / Unsupported]
- **Adaptations:** [Next.js overrides applied / JS JSDoc path / none]

### Design Fidelity (audit mode)
- **Status:** Not evaluated in audit mode (code-only scan). Refactor mode preserves design intent per Core Constraints.
- **Code-level layout risks:** [Only if grep finds DOM-breaking patterns, wrapper changes proposed, etc. — else "None identified"]

### Flagged Interactions (code-level scan)
| File:Line | Pattern | Reason |
|-----------|---------|--------|
| `src/components/X.tsx:88` | `stiffness: 800` | Likely jank — spring too stiff |

Or: `None found after code scan.`

### Dimension 8 — Reinvented Primitives (replace)
| Component | File:Line | Suggested Replacement |
|-----------|-----------|----------------------|

### Dimension 8 — Genuine Custom (preserve)
| Component | File:Line | Why Custom |
|-----------|-----------|------------|

### Violations
| # | Dimension | Issue | Severity | Evidence |
|---|-----------|-------|----------|----------|
| 1b | Reusability Pass | [plain language issue + why severity] | High/Medium/Low | `path:line` or grep — distinct per row |

### Vibecode Smells Detected
- **[Smell Name]:** [File:line locations + details]

### Grep Evidence Summary
- [Required: summarize output from grep patterns below]

### Recommended Fix Order
1. **[Dimension #] - [Fix in plain language]**: Why first.

### Dimensions Passed
- **Dimension [X] - [Name]**: Why it passed (with evidence).

### Dimensions Unevaluated
- **Dimension [X] - [Name]**: Why scan was not performed.
```

## Checklist

```
Stack detection (do this FIRST):
- [ ] Stack identified — React+TS, React+JS, Next.js, or redirect if unsupported

Pre-checks (audit mode):
- [ ] Design Fidelity — do NOT use "AT RISK"; state "Not evaluated in audit mode" unless code-level DOM/animation risks found
- [ ] Flagged Interactions — code scan with distinct file:line per finding, or explicitly "None found"
- [ ] Plain language — all Issue/Recommended Fix text rewritten for non-technical users

Handoff Audit Progress:
- [ ] 1.  Component Architecture — logic/presentation split; god-components broken; DOM hierarchy guard
- [ ] 1b. Reusability Pass — SEPARATE ROW ALWAYS — duplicate exports, similar filenames, repeated prop shapes; never merge into dim 1
- [ ] 2.  Data Extraction — MOCK_/seed data in /src/data/, not inline in UI components
- [ ] 3.  Canonical Domain Types — domain.ts/js as single source of truth; JS: evaluate JSDoc quality not just presence
- [ ] 4.  API Contract Stubs — all data via api.ts/js; @backend annotations; no direct fetch in components. ALL OR NOTHING
- [ ] 5.  State Management — context/reducers; no 5+ useState chains
- [ ] 6.  Read-Only / RBAC — guarded setters; inert for read-only roles
- [ ] 7.  Backend Integration Markers — @backend matches domain; BACKEND_CONTRACT.md sync
- [ ] 8.  Component Library Compliance — BOTH lists required (reinvented + genuine custom); unevaluated if scan skipped
- [ ] 9.  CSS Token Compliance — colors, spacing, sizing, typography; no inline hardcoded px/rem/em
- [ ] 10. Destructive Action Safety — ConfirmDialog or Toast+Undo
- [ ] 11. Routing & Navigation — react-router-dom or Next.js file routes
- [ ] 12. Component Performance — useMemo, useCallback, React.memo
- [ ] 13. Accessibility — semantic HTML, ARIA, keyboard nav, aria-live
- [ ] 14. Linting & Code Quality — TS: no any; JS: PropTypes + quality JSDoc
- [ ] 15. Error Boundaries & Resilience — boundaries, toast, loading skeletons
- [ ] 16. QA Test Selectors — data-testid on primary interactive elements
- [ ] 17. Dependency & Environment Hygiene — no unused deps, .env.example
- [ ] 18. File Hygiene & Icons — no orphans; SVG paths >10 lines extracted
```

## Evidence Rules

Every dimension marked as failing must include at least one specific **`path:line`** or grep output proving the violation.

- **Banned:** citing the same file twice as two different evidence entries without different line numbers and separate explanations.
- **Banned:** merging dimension 1b findings into dimension 1.
- **Required:** grep patterns below must be run (or equivalent regex) and summarized in Grep Evidence Summary.
- **Severity required:** every violation row includes High/Medium/Low with one-line justification (see Severity Scale in SKILL.md).

---

## Quick grep patterns

Run these to spot common violations. **Required in audit mode — include output in the report.**

```bash
# === STACK DETECTION ===
rg '"next"' package.json && echo "Next.js detected" || echo "Not Next.js"
find src -name '*.tsx' -o -name '*.ts' | head -3
find src -name '*.jsx' -o -name '*.js' | head -3

# === DIMENSION 1: ARCHITECTURE ===
find src \( -name '*.tsx' -o -name '*.jsx' \) | xargs -I {} awk 'END{if(NR>400)print FILENAME, NR}' {}

# === DIMENSION 1b: REUSABILITY (separate — do not merge with dim 1) ===
rg "^export (default )?(function|const) \w+" src/components src/pages --glob '*.{tsx,jsx}' -o | sort | uniq -d
find src/components src/pages -iname '*button*' -o -iname '*card*' -o -iname '*badge*' -o -iname '*input*' | sort
rg "variant.*size|size.*variant" src/components --glob '*.{tsx,jsx}' -l
rg -o 'className="[^"]{60,}"' src --glob '*.{tsx,jsx}' | sort | uniq -d

# === DIMENSION 2 & 4: DATA FLOW ===
rg "MOCK_|from.*\/data\/" src/components src/pages --glob '*.{tsx,ts,jsx,js}'
rg "const \w+ = \[" src/components src/pages --glob '*.{tsx,ts,jsx,js}'
rg "fetch\(|axios\.|\.get\(|\.post\(" src/components src/pages --glob '*.{tsx,ts,jsx,js}'

# === DIMENSION 3: DOMAIN TYPES / JSDOC QUALITY ===
rg "@typedef" src/domain.js 2>/dev/null || echo "No domain.js typedefs"
rg "interface |type " src/domain.ts 2>/dev/null || echo "No domain.ts interfaces"
rg "@typedef" src --glob '*.{js,jsx}' | wc -l
rg "@property" src/domain.js 2>/dev/null

# === DIMENSION 7: BACKEND MARKERS ===
rg "export (async )?function" src/api.ts src/api.js 2>/dev/null | rg -v "@backend"
rg "localhost|127\.0\.0\.1|http://|https://" src --glob '*.{tsx,ts,jsx,js}' --glob '!*.test.*'

# === DIMENSION 8: COMPONENT LIBRARY ===
rg -i "dropdown|modal|tooltip|popover|datepicker|date-picker|tablist|select.*option" src/components --glob '*.{tsx,jsx}' | rg -v "node_modules|shadcn|radix|headlessui|@radix"

# === DIMENSION 9: CSS TOKENS (colors, spacing, sizing, typography) ===
rg "#[0-9a-fA-F]{3,8}|rgb\(|hsl\(" src --glob '*.{tsx,jsx,css,scss}'
rg "style=\{\{" src --glob '*.{tsx,jsx}'
rg "fontSize:|padding:|margin:|width:|height:|lineHeight:|letterSpacing:|fontWeight:" src --glob '*.{tsx,jsx}'
rg "\d+px" src --glob '*.{tsx,jsx}' | rg "style="
rg "(m|p|gap|w|h|text|leading|tracking)-\[" src --glob '*.{tsx,jsx}'
rg "\[#[0-9a-fA-F]{3,8}\]|\[[\d.]+px\]" src --glob '*.{tsx,jsx}'

# === DIMENSION 10: DESTRUCTIVE ACTIONS ===
rg "onClick=\{.*delete|\.remove\(|reset\(" src --glob '*.{tsx,jsx}'

# === DIMENSION 5 & 11: STATE & ROUTING ===
rg "page === |currentPage|setPage\(" src --glob '*.{tsx,jsx}'
rg -l "useState" src --glob '*.{tsx,jsx}' | xargs -I {} sh -c 'count=$(rg -c "useState" {}); [ "$count" -ge 5 ] && echo "{}: $count useState calls"'

# === DIMENSION 14: TYPING ===
rg ":\s*any\b|as any" src --glob '*.{tsx,ts,jsx,js}'

# === DIMENSION 16: QA SELECTORS ===
rg -L "data-testid" src/components --glob '*.{tsx,jsx}'

# === DIMENSION 17 & 18: HYGIENE ===
npx depcheck --skip-missing 2>/dev/null
rg "console\.(log|warn|error)" src --glob '*.{tsx,ts,jsx,js}' --glob '!*.test.*'
rg -l "<svg" src/components src/pages --glob '*.{tsx,jsx}'

# === FLAGGED INTERACTIONS ===
rg "stiffness:\s*[5-9]\d{2,}|damping:\s*[0-4]\b" src --glob '*.{tsx,jsx}'
rg "transition.*0ms|transition.*0s[^0-9]|duration-0" src --glob '*.{tsx,jsx,css,scss}'
rg "setTimeout|setInterval" src/components src/pages --glob '*.{tsx,jsx}'
rg "onMouse|onHover|onFocus.*set[A-Z]" src/components --glob '*.{tsx,jsx}'
```

---

## Vibecode smell quick-check

| Smell | Grep Command |
|-------|---------|
| God component (400+ lines) | `find src \( -name '*.tsx' -o -name '*.jsx' \) \| xargs -I {} awk 'END{if(NR>400)print FILENAME,NR}' {}` |
| Duplicate component exports (1b) | `rg "^export (default )?(function\|const) \w+" src/components --glob '*.{tsx,jsx}' -o \| sort \| uniq -d` |
| Similar button/card filenames (1b) | `find src -iname '*button*' -o -iname '*card*'` |
| Direct mock import | `rg "MOCK_\|from.*\/data\/" src/components src/pages` |
| Third-party direct fetch | `rg "fetch(\|axios\." src/components src/pages` |
| Reinvented primitive | `rg -i "dropdown\|modal\|tooltip" src/components \| rg -v "radix\|shadcn"` |
| Inline style hardcoding (9) | `rg "style=\{\{" src --glob '*.{tsx,jsx}'` |
| Hardcoded fontSize/padding (9) | `rg "fontSize:\|padding:\|margin:" src --glob '*.{tsx,jsx}'` |
| Tailwind arbitrary spacing (9) | `rg "(m\|p\|gap\|w\|h)-\[" src --glob '*.{tsx,jsx}'` |
| Weak JSDoc (3, JS mode) | `rg "@typedef" src/domain.js \| wc -l` vs entities used in UI |
| Missing @backend | `rg "export (async )?function" src/api.ts src/api.js 2>/dev/null \| rg -v "@backend"` |
| Suspicious animation | `rg "stiffness:\s*[5-9]\d{2,}" src --glob '*.{tsx,jsx}'` |

---

## Target file layout (handoff-ready)

```
src/
├── data/               # Extracted mock/seed data (never imported directly by components — always via api.ts/js)
├── domain.ts           # or domain.js — canonical business types
├── api.ts              # or api.js — typed async stubs with @backend annotations
├── components/
│   ├── ui/             # Shared reusable primitives — one per pattern
│   ├── icons/          # Extracted SVG icon components
│   └── ConfirmDialog.tsx
├── contexts/           # One isolated context provider per domain
├── hooks/              # Shared custom hooks
├── routes/             # react-router setup + guards (or Next.js app/ pages/)
└── utils/              # Helpers, formatters
.env.example
BACKEND_CONTRACT.md
```
