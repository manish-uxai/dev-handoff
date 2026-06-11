# 20-Dimension Audit Checklist

Copy this checklist when auditing a vibe-coded prototype for production readiness.

```
Pre-flight:
- [ ] Run `node --version` — if Node.js missing, stop and direct designer to install it
- [ ] If Node present, continue

Step 0:
- [ ] Discovery pass — map the project purpose, stack, conventions before judging anything
- [ ] Stack detected — React/TS, React/JS, Next.js, or Plain HTML
  - React/TS → proceed with 20 dimensions
  - React/JS → note "will be migrated to TS as first refactor step" (don't flag missing PropTypes)
  - Plain HTML → note "will be converted to Vite + React + TS in refactor mode"
  - Unsupported framework → stop and redirect

Visual fidelity note:
- [ ] State once at top: "Visual fidelity should be verified in the browser after any refactoring." Do not mark as AT RISK.

Flagged Interactions (run interaction smell greps below):
- [ ] List findings with file + line number, or explicitly state "No interaction smells found" — skipping this section is not acceptable
- [ ] When citing the same file for multiple issues, include distinct line numbers and describe what is different

Audit Progress:
- [ ] 1.  Component Architecture — god-components broken; stateful/presentational split; no circular imports
- [ ] 1b. Reusability Pass (SEPARATE step) — scan entire codebase for: similarly named component files across folders, overlapping className patterns, copy-pasted JSX structures, files serving same purpose; list every pattern with file locations; if not performed mark INCOMPLETE
- [ ] 2.  Data Extraction — MOCK_/seed data in /src/data/, not inline in components
- [ ] 3.  Canonical Domain Types — domain.ts with complete, non-duplicated TypeScript interfaces matching actual runtime usage
- [ ] 4.  API Contract Stubs — all data through api.ts; all stubs carry @backend annotations; NO component calls fetch() or imports data directly (ALL OR NOTHING — any direct fetch = fail)
- [ ] 5.  State Management — context/reducers; no 5+ useState chains; ANY new provider must be mounted in the tree (verify at runtime, not just build)
- [ ] 6.  Read-Only / RBAC (CONDITIONAL) — only if app has roles/read-only; if not, mark N/A, don't flag absence
- [ ] 7.  Backend Integration Markers — @backend annotations in code match domain.ts shapes
- [ ] 8.  Component Library Compliance & Reuse — reuse existing component first, else import shadcn/Radix, never hand-roll a duplicate; consolidate duplicate components serving the same purpose; ACTIVELY SCAN, if no scan mark UNEVALUATED
- [ ] 9.  Design Token Compliance — no hardcoded colors, spacing, sizing, typography, or raw pixels in inline styles, CSS, or Tailwind arbitrary values
- [ ] 10. Missing Cases (Destructive/Empty/Error) — ConfirmDialog gates destructive actions; every list/table has empty state; every fallible action has error state (the cases vibecoding forgets)
- [ ] 11. Routing & Navigation (CONDITIONAL) — single-page: skip. Multi-page with no router: force react-router. Multi-page with a router: leave it.
- [ ] 12. Expensive Operations (NARROW) — do NOT add useMemo/useCallback as blanket practice (anti-pattern). Only flag genuinely expensive ops that visibly lag with real data. Usually passes.
- [ ] 13. Accessibility & Readability — CHECK CONTRAST FIRST (dark-on-dark / light-on-light unreadable text); then semantic HTML, ARIA, keyboard nav, focus
- [ ] 14. Code Quality (TOP-TIER) — TypeScript only: zero any types, no @ts-ignore/as any, strict interfaces, organized imports, no dead code, consistent patterns
- [ ] 15. Error Boundaries & Component Resilience — route-level error boundaries; PLUS every data-consuming component handles loading, error, and empty states
- [ ] 16. Dependency, Environment & Onboarding — no unused deps, pinned versions, .env.example present, README has install+run instructions
- [ ] 17. File Hygiene & Icons — no orphans, unused imports, dead code; raw/inline SVGs (esp. from Figma MCP) replaced with real library icons
- [ ] 18. Component Production Readiness — spot-check major components for: null/undefined handling, text overflow, empty array behavior, baked-in UI text, prop interface clarity, style isolation
- [ ] 19. Security Basics — no hardcoded secrets, no dangerouslySetInnerHTML with unsanitized data, npm audit clean, .env not committed
- [ ] 20. Design Quality — no AI-slop (decorative accent borders, one-off colors), all colors mapped to tokens, severity separate from metric color in charts

Output quality:
- [ ] Every finding pairs a technical term with a plain consequence (term teaches, consequence clarifies)
- [ ] Security and design-system findings in full clear prose, never compressed
- [ ] Every finding labeled as FACT (file:line proof) or JUDGMENT (reasoned opinion)
- [ ] "Summary for the designer" section at end — warm, plain, no jargon
- [ ] Self-review gate passed: citations have line numbers, no padding, numbers verified, summary jargon-free
- [ ] Prefer 15 high-confidence findings over 50 speculative ones
- [ ] Trust grep output — don't over-read files to double-check what grep already proved
```

---

## Evidence Rules

Every failing dimension must include specific file paths WITH line numbers. Not opinions, not filenames alone.

**Citation format (non-negotiable):**

- Every citation: `file.jsx:42` not just `file.jsx`
- Same file cited twice = ONLY if different line numbers AND different issues described
- `api.js, api.js, api.js` is NEVER acceptable. Write `api.js:42 (issue A), api.js:67 (issue B)` instead.
- If line number unknown: write `file.jsx:? (line not verified)` — honest and acceptable
- Every finding labeled as **FACT** (file:line proof) or **JUDGMENT** (reasoned opinion)

**Number verification:**

- File line counts, useState counts, or any specific number must come from actual grep/awk output — not estimation
- If you didn't run the count, don't state a specific number. Say "very large file" instead of "23,053 lines"

**Severity required:** every finding must include High/Medium/Low with one sentence explaining why.

The grep patterns below are **required** evidence-gathering steps. Run them and include output summaries.

---

## Quick grep patterns

**Required in audit mode. Include output in the report.**

**Run these in small batches (one dimension group at a time), not as one giant chained command.** A single massive command produces output that gets truncated, forcing re-runs that waste tokens. Run 3-4 related greps, read the output, move to the next group. Each `# ===` section below is a natural batch boundary.

```bash
# === PRE-FLIGHT: NODE.JS CHECK ===
node --version || echo "NODE_MISSING — stop and direct designer to install"

# === STEP 0: STACK DETECTION ===

# Check for package.json (no package.json = likely plain HTML)
ls package.json 2>/dev/null && echo "package.json found" || echo "NO_PACKAGE_JSON — likely plain HTML project"

# Check for React/TS/JS files
find src -name '*.jsx' -o -name '*.js' | head -5
find src -name '*.tsx' -o -name '*.ts' | head -5

# Check for HTML-only (no React)
find . -maxdepth 2 -name '*.html' | head -10

# === DIMENSION 1: ARCHITECTURE ===

# God components (400+ lines)
find src \( -name '*.tsx' -o -name '*.jsx' \) | xargs -I {} awk 'END{if(NR>400)print FILENAME, NR}' {}

# Duplicated component class names (reusability smell)
grep -RhoE 'className="[^"]{60,}"' src --include='*.jsx' --include='*.tsx' | sort | uniq -d | head -40

# Similarly named component files across folders
find src -name '*.jsx' -o -name '*.tsx' | xargs -I {} basename {} | sort | uniq -d

# Relative import depth (should use @/* aliases)
grep -RInE '\.\./\.\.' src --include='*.jsx' --include='*.tsx' --include='*.ts' --include='*.js' | head -40

# Circular dependency candidates (files importing each other)
for f in $(find src -name '*.jsx' -o -name '*.tsx' | head -50); do
  imports=$(grep -oE "from ['\"]\..*['\"]" "$f" | sed "s/from ['\"]//;s/['\"]//")
  for imp in $imports; do
    target=$(find src -name "$(basename "$imp")*" 2>/dev/null | head -1)
    if [ -n "$target" ] && grep -q "$(basename "$f" | sed 's/\..*//')" "$target" 2>/dev/null; then
      echo "CIRCULAR: $f <-> $target"
    fi
  done
done | head -20

# === DIMENSION 2 & 4: DATA FLOW ===

# Direct mock data imports inside components
grep -RInE 'MOCK_|from.*/data/' src/components src/pages --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | head -80

# Inline mock arrays in components
grep -RInE 'const\s+\w+\s*=\s*\[' src/components src/pages --include='*.jsx' --include='*.tsx' | head -80

# Components fetching directly (dimension 4 — all or nothing)
grep -RInE 'fetch\(|axios\.|\.get\(|\.post\(' src/components src/pages --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | head -80

# === DIMENSION 7: BACKEND MARKERS ===

# Missing @backend annotations in api file
grep -nE 'export\s+(async\s+)?function' src/api.ts src/api.js src/services/api.ts src/services/api.js 2>/dev/null | grep -v '@backend'

# Hardcoded API URLs
grep -RInE 'localhost|127\.0\.0\.1|http://|https://' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' --exclude='*.test.*' | head -80

# === DIMENSION 8: COMPONENT LIBRARY COMPLIANCE ===

# Reinvented UI primitives
grep -RInEi 'dropdown|modal|tooltip|popover|datepicker|date-picker|tablist|select.*option' src/components --include='*.jsx' --include='*.tsx' | grep -Evi 'node_modules|shadcn|radix|headlessui|/ui/' | head -80

# === DIMENSION 9: DESIGN TOKEN COMPLIANCE ===

# Hardcoded hex colors in JSX/CSS
grep -RInE '#[0-9a-fA-F]{3,8}|rgb\(|hsl\(' src --include='*.jsx' --include='*.tsx' --include='*.css' --include='*.scss' | head -80

# Tailwind arbitrary color values
grep -RInE '\[#[0-9a-fA-F]{3,8}\]' src --include='*.jsx' --include='*.tsx' | head -40

# Tailwind arbitrary spacing/sizing values
grep -RInE '\[[0-9.]+px\]|\[[0-9.]+rem\]|\[[0-9.]+em\]' src --include='*.jsx' --include='*.tsx' | head -80

# Inline style hardcoded pixels
grep -RInE "style=\{\{[^}]*(padding|margin|width|height|fontSize|gap|top|left|right|bottom|borderRadius|lineHeight)[^}]*[0-9]+px" src --include='*.jsx' --include='*.tsx' | head -80

# Inline style hardcoded pixels (string format)
grep -RInE "style=\{\{[^}]*(padding|margin|width|height|fontSize|gap|borderRadius|lineHeight)[^}]*'[0-9]+" src --include='*.jsx' --include='*.tsx' | head -80

# CSS raw pixel values (non-Tailwind)
grep -RInE '(padding|margin|width|height|font-size|gap|border-radius|line-height):\s*[0-9]+px' src --include='*.css' --include='*.scss' | head -80

# === DIMENSION 5 & 11: STATE & ROUTING ===

# useState chains (5+ in one file)
find src \( -name '*.jsx' -o -name '*.tsx' \) | while read f; do c=$(grep -c 'useState' "$f"); if [ "$c" -ge 5 ]; then echo "$f: $c useState calls"; fi; done | sort -t: -k2 -nr | head -30

# Provider-mount safety: list providers defined, then check each is mounted (consumed-but-not-wrapped = runtime crash)
grep -RIn 'Provider' src --include='*.tsx' --include='*.jsx' | grep -iE 'createContext|<.*Provider>|Provider =' | head -30

# Routing: single vs multi-page, and conditional-render anti-pattern (only force router if multi-page + no router)
grep -RInE 'page === |currentPage|setPage\(|activeTab|activeView' src --include='*.jsx' --include='*.tsx' | head -40
grep -RInE 'react-router-dom|BrowserRouter|Routes|Route|useNavigate' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | head -40

# === DIMENSION 12: EXPENSIVE OPERATIONS (narrow — do NOT mandate useMemo) ===
# Only flag genuinely heavy ops in hot paths; missing memoization is NOT a finding.
grep -RInE '\.(sort|filter|map|reduce)\([^)]*\)\s*\.(sort|filter|map|reduce)' src/components --include='*.tsx' | head -20

# === DIMENSION 14: CODE QUALITY (top-tier) ===

# any types and escape hatches (must be eradicated)
grep -RInE ':\s*any\b|as any|@ts-ignore|@ts-nocheck' src --include='*.tsx' --include='*.ts' | head -40

# JSX/JS files present = migration to TS required first (HARD GATE, not a PropTypes finding)
echo "jsx_js_files=$(find src -name '*.jsx' -o -name '*.js' | grep -v test | wc -l | tr -d ' ') (if > 0, migrate to TS as MANDATORY first step before any other dimension)"

# === DIMENSION 15: RESILIENCE ===

# Error boundaries
grep -RInE 'ErrorBoundary|componentDidCatch|getDerivedStateFromError' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | head -40

# Loading/empty state patterns
grep -RInE 'isLoading|loading|Skeleton|Spinner|emptyState|empty-state|no data|no results' src --include='*.jsx' --include='*.tsx' | head -80

# === DIMENSION 10: MISSING CASES (DESTRUCTIVE / EMPTY / ERROR) ===

# Destructive actions — check they're gated by a confirm
grep -RInE 'onClick=\{[^}]*(delete|remove|reset|discard|clear)' src/components src/containers --include='*.jsx' --include='*.tsx' --include='*.ts' | head -30
grep -RIn 'ConfirmDialog\|confirm(' src --include='*.jsx' --include='*.tsx' --include='*.ts' | head -20

# Empty/error state presence (low count relative to data views = likely missing)
grep -RInE 'empty|no data|no results|isError|hasError' src/components --include='*.jsx' --include='*.tsx' --include='*.ts' | wc -l

# === DIMENSION 13: ACCESSIBILITY & READABILITY ===

# Contrast risk — same-tone bg+text pairings (dark-on-dark, light-on-light)
grep -RInE 'bg-(black|gray-9|slate-9|zinc-9|neutral-9).*text-(black|gray-9|slate-9|zinc-9)|bg-(white|gray-1|slate-1).*text-(white|gray-1|slate-1)' src --include='*.jsx' --include='*.tsx' | head -20
# Manual review still needed — grep only catches obvious Tailwind pairings, not token-based ones

# Semantic HTML + ARIA
grep -RInE '<nav|<main|<header|<aside|aria-label|aria-live|role=' src/components src/containers --include='*.jsx' --include='*.tsx' | wc -l

# === DIMENSION 16 & 17: HYGIENE ===

# Unused dependencies
npx depcheck --skip-missing 2>/dev/null

# Console statements in production code
grep -RInE 'console\.(log|warn|error)' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' --exclude='*.test.*' | head -80

# Raw/inline SVGs (esp. from Figma MCP — often garbled icons; replace with library icons)
grep -RIl '<svg' src/components src/pages --include='*.jsx' --include='*.tsx' | head -40

# .env.example check
ls -la .env.example 2>/dev/null || echo '.env.example missing'

# README setup instructions check
if [ -f README.md ]; then
  echo "README.md exists"
  grep -qiE 'npm install|yarn install|pnpm install' README.md && echo "has install instructions" || echo "MISSING install instructions"
  grep -qiE 'npm run dev|yarn dev|pnpm dev|npm start' README.md && echo "has run instructions" || echo "MISSING run instructions"
else
  echo "README.md MISSING"
fi

# === DIMENSION 19: SECURITY BASICS ===

# Hardcoded API keys / tokens / secrets in source
grep -RInE 'sk_live|pk_live|sk_test|pk_test|AKIA[0-9A-Z]{16}|ghp_[a-zA-Z0-9]{36}|api_key\s*[:=]\s*["\x27][a-zA-Z0-9]' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | head -20

# Firebase/Stripe/Auth config with hardcoded keys
grep -RInE 'apiKey|authDomain|projectId|storageBucket|messagingSenderId|appId|measurementId' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | grep -v 'process\.env\|import\.meta\.env' | head -20

# dangerouslySetInnerHTML usage
grep -RIn 'dangerouslySetInnerHTML' src --include='*.jsx' --include='*.tsx' | head -20

# eval or new Function
grep -RInE 'eval\(|new Function\(' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | head -20

# .env committed (should be in .gitignore)
ls -la .env 2>/dev/null && echo 'WARNING: .env file exists — check .gitignore'
grep -q '\.env' .gitignore 2>/dev/null && echo '.env is in .gitignore' || echo 'WARNING: .env NOT in .gitignore'

# npm audit for known vulnerabilities (CONDITIONAL — skip if it hangs or environment is offline)
# This can be slow and verbose. Run it with a timeout. If it doesn't return quickly, note "run npm audit manually" and move on.
timeout 30 npm audit --audit-level=high 2>/dev/null | tail -15 || echo "npm audit skipped — run manually before handoff"

# === DIMENSION 20: DESIGN QUALITY (AI SLOP) ===

# Decorative accent borders on cards (AI-dashboard cliché)
grep -RInE 'border-l-[0-9]|border-r-[0-9]|borderLeft|borderRight' src/components --include='*.jsx' --include='*.tsx' | grep -iE 'card' | head -20

# One-off colors not using tokens (raw Tailwind color classes instead of var tokens)
grep -RInE 'bg-(red|blue|green|gray|slate|zinc|yellow|orange|purple|pink)-[0-9]|text-(red|blue|green|gray|slate|zinc)-[0-9]' src/components --include='*.jsx' --include='*.tsx' | head -40

# Inline color styles (should be tokens)
grep -RInE "style=\{\{[^}]*color[^}]*#" src/components --include='*.jsx' --include='*.tsx' | head -20

# === DIMENSION 18: PRODUCTION READINESS ===

# Components with no null/undefined guards
grep -RInE 'props\.\w+\.' src/components --include='*.jsx' --include='*.tsx' | grep -v '?.' | grep -v '&&' | head -40

# Hardcoded labels/placeholders inside components
grep -RInE "placeholder=['\"][A-Z]|label=['\"][A-Z]|<p>[A-Z][a-z].*</p>|<span>[A-Z][a-z].*</span>|<h[1-6]>[A-Z]" src/components --include='*.jsx' --include='*.tsx' | head -80

# Fixed pixel widths that prevent responsive behavior
grep -RInE 'width:\s*[0-9]{3,}px|w-\[[0-9]{3,}px\]|minWidth:\s*[0-9]{3,}' src/components --include='*.jsx' --include='*.tsx' | head -40

# === FLAGGED INTERACTIONS ===

# Extreme Framer Motion spring configs
grep -RInE 'stiffness:\s*[5-9][0-9]{2,}|damping:\s*[0-4]\b' src --include='*.jsx' --include='*.tsx' | head -40

# Zero-duration transitions
grep -RInE 'transition.*0ms|transition.*0s[^0-9]|duration-0' src --include='*.jsx' --include='*.tsx' --include='*.css' --include='*.scss' | head -40

# setTimeout/setInterval driving visuals
grep -RInE 'setTimeout|setInterval' src/components src/pages --include='*.jsx' --include='*.tsx' | head -80

# Hover/focus handlers toggling state directly
grep -RInE 'onMouse|onHover|onFocus.*set[A-Z]' src/components --include='*.jsx' --include='*.tsx' | head -80
```

---

## Vibecode smell quick-check

| Smell                              | Grep                                                                                                                  |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| God component (400+ lines)         | `find src \( -name '*.tsx' -o -name '*.jsx' \) \| xargs -I {} awk 'END{if(NR>400)print FILENAME,NR}' {}`              |
| Duplicate component names          | `find src -name '*.jsx' -o -name '*.tsx' \| xargs -I {} basename {} \| sort \| uniq -d`                               |
| Direct mock import                 | `grep -RInE 'MOCK_\|from.*/data/' src/components src/pages --include='*.jsx'`                                         |
| Direct fetch in component          | `grep -RInE 'fetch\(\|axios\.' src/components src/pages --include='*.jsx'`                                            |
| Reinvented primitive               | `grep -RInEi 'dropdown\|modal\|tooltip\|popover' src/components --include='*.jsx' \| grep -Evi 'radix\|shadcn\|/ui/'` |
| Hardcoded hex in Tailwind          | `grep -RInE '\[#[0-9a-fA-F]{3,8}\]' src --include='*.jsx'`                                                            |
| Hardcoded spacing in Tailwind      | `grep -RInE '\[[0-9.]+px\]' src --include='*.jsx'`                                                                    |
| Inline style raw pixels            | `grep -RInE "style=\{\{[^}]*(padding\|margin\|width\|height\|fontSize)[^}]*[0-9]+px" src --include='*.jsx'`           |
| Missing loading/empty states       | `grep -RIL 'isLoading\|loading\|Skeleton\|Spinner\|empty' src/components --include='*.jsx'`                           |
| No null guards                     | `grep -RInE 'props\.\w+\.' src/components --include='*.jsx' \| grep -v '?.' \| head -40`                              |
| Hardcoded labels                   | `grep -RInE "placeholder=['\"][A-Z]\|label=['\"][A-Z]" src/components --include='*.jsx'`                              |
| Missing @backend annotations       | `grep -nE 'export\s+(async\s+)?function' src/api.js src/services/api.js 2>/dev/null \| grep -v '@backend'`            |
| Interaction: setTimeout visuals    | `grep -RInE 'setTimeout\|setInterval' src/components --include='*.jsx'`                                               |
| Fixed widths preventing responsive | `grep -RInE 'width:\s*[0-9]{3,}px\|w-\[[0-9]{3,}px\]' src/components --include='*.jsx'`                               |
| Hardcoded API keys                 | `grep -RInE 'sk_live\|pk_live\|AKIA\|api_key\s*[:=]' src --include='*.jsx' --include='*.js'`                          |
| dangerouslySetInnerHTML            | `grep -RIn 'dangerouslySetInnerHTML' src --include='*.jsx' --include='*.tsx'`                                         |
| README missing setup               | `grep -qiE 'npm install\|npm run dev' README.md \|\| echo 'MISSING'`                                                  |
| Circular imports                   | Check if file A imports B and B imports A                                                                             |
| Relative deep imports              | `grep -RInE '\.\./\.\.' src --include='*.jsx' --include='*.tsx' \| head -20`                                          |

---

## Target file layout

See SKILL.md "Target File Layout" section — single source of truth, not duplicated here to avoid drift.
