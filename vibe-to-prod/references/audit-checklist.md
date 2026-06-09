# 18-Dimension Handoff Audit Checklist

Copy this checklist when auditing a vibecoded prototype for backend handoff.

## Audit output format

When running `/vibe-to-prod audit`, output must strictly follow:

```markdown
## Handoff Audit: [Scope / Module Name]

### Language Check (Blocker)
- **Status:** [TypeScript OK / JavaScript codebase — dimension 14 blocker]
- **Impact:** [If JS: dimensions 3, 4, 6, 7, 14 structurally compromised; recommend TS migration]

### Design Intent & Fidelity Status
- **Status:** [OK / AT RISK — layout, micro-interactions, animation feel]

### Flagged Interactions (code-level scan)
- [List suspicious animation/hover/timeout patterns with file paths and reasoning, OR state "None found after code scan"]

### Violations
| # | Dimension | Issue | Severity | Evidence |
|---|-----------|-------|----------|----------|
| [1-18] | [Dimension Name] | [Specific violation] | [High / Medium / Low] | `path:line` or grep output |

### Vibecode Smells Detected
- **[Smell Name]:** [File locations + details]

### Grep Evidence Summary
- [Required: summarize output from grep patterns below, or equivalent regex results]

### Recommended Fix Order
1. **[Dimension #] - [Fix]**: Why first.

### Dimensions Passed
- **Dimension [X] - [Name]**: Why it passed (with evidence).

### Dimensions Unevaluated
- **Dimension [X] - [Name]**: Why scan was not performed (e.g., dim 8 without primitive scan).
```

## Checklist

```
Blocker Check (do this FIRST, report at top of audit output):
- [ ] Language Check — if codebase is .js/.jsx instead of .ts/.tsx, flag as dimension 14 blocker; dimensions 3, 4, 6, 7, 14 are structurally compromised

Pre-checks:
- [ ] Design Surface Fidelity — layout, spacing, animations, interactions render identically before and after
- [ ] Implementation Improvements — any animation/interaction refactors preserve perceived feel, not just code shape
- [ ] Flagged Interactions — scan code for interaction smells even without browser (see patterns below); list findings or explicitly state none found; do NOT skip this section

Handoff Audit Progress:
- [ ] 1.  Component Architecture — logic/presentation split; god-components broken
- [ ] 1b. Reusability Pass (separate step) — duplicated UI patterns identified with file locations; consolidated into shared primitives in components/ui/
- [ ] 2.  Data Extraction — MOCK_/seed data in /src/data/, not inline in UI components
- [ ] 3.  Canonical Domain Types — domain.ts as strict TypeScript API contract
- [ ] 4.  API Contract Stubs — typed async api.ts with simulated latency; all stubs carry @backend annotations; no component imports mock data directly; no component calls fetch() directly (including maps, charts, third-party visuals). ALL OR NOTHING — any direct fetch = fail
- [ ] 5.  State Management — context/reducers with loading/error/optimistic states
- [ ] 6.  Read-Only / RBAC — guarded setters; inert attribute for read-only roles
- [ ] 7.  Backend Integration Markers — @backend annotations match domain.ts; BACKEND_CONTRACT.md generated
- [ ] 8.  Component Library Compliance — ACTIVELY SCAN for reinvented primitives; produce two lists: (a) reinvented primitives with file locations, (b) genuine custom components to preserve. If no scan performed, mark UNEVALUATED not passing
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

## Evidence Rules

Every dimension marked as failing must include at least one specific file path, line reference, or grep output proving the violation. The audit is evidence-based, not opinion-based.

- Citing the same file twice as two different sources is padding, not evidence. Each citation must point to a distinct location or finding.
- The grep patterns below are **required evidence-gathering steps**, not optional. Run them and include the output (or a summary) in the audit report.
- If the environment doesn't support shell execution, run equivalent regex searches and show results.

---

## Quick grep patterns

Run these to spot common violations. **These are required in audit mode — include output in the report.**

```bash
# === BLOCKER CHECK (run first) ===

# Language check — are we in JS or TS?
find src -name '*.jsx' -o -name '*.js' | head -5 && echo "--- JS files found: flag as dimension 14 blocker ---"
find src -name '*.tsx' -o -name '*.ts' | head -5 && echo "--- TS files found ---"

# === DIMENSION 1: ARCHITECTURE ===

# God components (files over 400 lines)
find src -name '*.tsx' -o -name '*.jsx' | xargs -I {} awk 'END{if(NR>400)print FILENAME, NR}' {}

# Duplicated component class names (reusability smell)
rg -o 'className="[^"]{60,}"' src --glob '*.{tsx,jsx}' | sort | uniq -d

# === DIMENSION 2 & 4: DATA FLOW ===

# Direct mock data imports inside components or pages (hard rule violation)
rg "MOCK_|from.*\/data\/" src/components src/pages --glob '*.{tsx,ts,jsx,js}'

# Inline mock arrays in components (not routed through api.ts)
rg "const \w+ = \[" src/components src/pages --glob '*.{tsx,ts,jsx,js}'

# Third-party components fetching directly (dimension 4 — all or nothing)
rg "fetch\(|axios\.|\.get\(|\.post\(" src/components src/pages --glob '*.{tsx,ts,jsx,js}'

# === DIMENSION 7: BACKEND MARKERS ===

# Missing @backend annotations in api.ts (or api.js)
rg "export (async )?function" src/api.ts src/api.js 2>/dev/null | rg -v "@backend"

# Hardcoded API URLs
rg "localhost|127\.0\.0\.1|http://|https://" src --glob '*.{tsx,ts,jsx,js}' --glob '!*.test.*'

# === DIMENSION 8: COMPONENT LIBRARY COMPLIANCE ===

# Reinvented UI primitives (candidates for shadcn/Radix replacement)
rg -i "dropdown|modal|tooltip|popover|datepicker|date-picker|tablist|select.*option" src/components --glob '*.{tsx,jsx}' | rg -v "node_modules|shadcn|radix|headlessui"

# === DIMENSION 9: CSS TOKENS ===

# Hardcoded colors
rg "#[0-9a-fA-F]{3,8}|rgb\(|hsl\(" src --glob '*.{tsx,jsx,css,scss}'

# Hardcoded Tailwind arbitrary values (candidates for CSS variables)
rg "\[#[0-9a-fA-F]{3,8}\]|\[[\d.]+px\]" src --glob '*.{tsx,jsx}'

# === DIMENSION 10: DESTRUCTIVE ACTIONS ===

# Destructive actions without confirm dialog
rg "onClick=\{.*delete|\.remove\(|reset\(" src --glob '*.{tsx,jsx}'

# === DIMENSION 5 & 11: STATE & ROUTING ===

# Conditional routing anti-pattern
rg "page === |currentPage|setPage\(" src --glob '*.{tsx,jsx}'

# useState chains (5+ in one component — reducer candidate)
rg -l "useState" src --glob '*.{tsx,jsx}' | xargs -I {} sh -c 'count=$(rg -c "useState" {}); [ "$count" -ge 5 ] && echo "{}: $count useState calls"'

# === DIMENSION 14: TYPING ===

# any types
rg ":\s*any\b|as any" src --glob '*.{tsx,ts,jsx,js}'

# === DIMENSION 16: QA SELECTORS ===

# Missing data-testid
rg -L "data-testid" src/components --glob '*.{tsx,jsx}'

# === DIMENSION 17 & 18: HYGIENE ===

# Unused dependencies
npx depcheck --skip-missing 2>/dev/null

# Console.log in production code
rg "console\.(log|warn|error)" src --glob '*.{tsx,ts,jsx,js}' --glob '!*.test.*'

# Inline SVG blocks (extract to icons/)
rg -l "<svg" src/components src/pages --glob '*.{tsx,jsx}'

# === FLAGGED INTERACTIONS (code-level, no browser needed) ===

# Framer Motion springs with extreme values (likely jank)
rg "stiffness:\s*[5-9]\d{2,}|damping:\s*[0-4]\b" src --glob '*.{tsx,jsx}'

# CSS transitions with 0ms/0s duration (likely missing or broken)
rg "transition.*0ms|transition.*0s[^0-9]|duration-0" src --glob '*.{tsx,jsx,css,scss}'

# setTimeout/setInterval driving visual changes (should use CSS or animation library)
rg "setTimeout|setInterval" src/components src/pages --glob '*.{tsx,jsx}'

# Hover/focus handlers toggling state directly (likely flicker without debounce)
rg "onMouse|onHover|onFocus.*set[A-Z]" src/components --glob '*.{tsx,jsx}'
```

---

## Vibecode smell quick-check

| Smell | Grep Command |
|-------|---------|
| JavaScript codebase (blocker) | `find src -name '*.jsx' -o -name '*.js' \| head -5` |
| God component (400+ lines) | `find src \( -name '*.tsx' -o -name '*.jsx' \) \| xargs -I {} awk 'END{if(NR>400)print FILENAME,NR}' {}` |
| Direct mock import in component | `rg "MOCK_\|from.*\/data\/" src/components src/pages --glob '*.{tsx,jsx}'` |
| Third-party direct fetch | `rg "fetch\(\|axios\." src/components src/pages --glob '*.{tsx,jsx}'` |
| Reinvented primitive | `rg -i "dropdown\|modal\|tooltip\|popover" src/components --glob '*.{tsx,jsx}' \| rg -v "radix\|shadcn"` |
| Duplicated component variants | `rg -o 'className="[^"]{60,}"' src --glob '*.{tsx,jsx}' \| sort \| uniq -d` |
| No TypeScript interfaces | `rg -L "interface\|type " src --glob '*.{tsx,jsx}'` |
| Missing loading states | `rg -L "loading\|isLoading\|skeleton\|Spinner" src --glob '*.{tsx,jsx}'` |
| Hardcoded hex in Tailwind | `rg "\[#[0-9a-fA-F]{3,8}\]" src --glob '*.{tsx,jsx}'` |
| Inline SVGs (10+ lines) | `rg -l "<svg" src/components src/pages --glob '*.{tsx,jsx}'` |
| Missing @backend annotations | `rg "export (async )?function" src/api.ts src/api.js 2>/dev/null \| rg -v "@backend"` |
| Missing data-testid | `rg -L "data-testid" src/components --glob '*.{tsx,jsx}'` |
| Suspicious animation configs | `rg "stiffness:\s*[5-9]\d{2,}\|damping:\s*[0-4]\b" src --glob '*.{tsx,jsx}'` |
| setTimeout driving visuals | `rg "setTimeout\|setInterval" src/components src/pages --glob '*.{tsx,jsx}'` |

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