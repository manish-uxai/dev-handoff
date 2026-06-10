# 20-Dimension Audit Checklist

Copy this checklist when auditing a vibe-coded prototype for production readiness.

```
Step 0:
- [ ] Discovery pass — map the project purpose, stack, conventions before judging anything
- [ ] Stack detected — React/TS, React/JS, or Next.js (if unsupported, stop and redirect)

Visual fidelity note:
- [ ] State once at top: "Visual fidelity should be verified in the browser after any refactoring." Do not mark as AT RISK.

Flagged Interactions (run interaction smell greps below):
- [ ] List findings with file + line number, or explicitly state "No interaction smells found" — skipping this section is not acceptable
- [ ] When citing the same file for multiple issues, include distinct line numbers and describe what is different

Audit Progress:
- [ ] 1.  Component Architecture — god-components broken; stateful/presentational split; no circular imports
- [ ] 1b. Reusability Pass (SEPARATE step) — scan entire codebase for: similarly named component files across folders, overlapping className patterns, copy-pasted JSX structures, files serving same purpose; list every pattern with file locations; if not performed mark INCOMPLETE
- [ ] 2.  Data Extraction — MOCK_/seed data in /src/data/, not inline in components
- [ ] 3.  Canonical Domain Types — domain.ts or domain.js with complete, non-duplicated type definitions matching actual runtime usage
- [ ] 4.  API Contract Stubs — all data through api.ts/api.js; all stubs carry @backend annotations; NO component calls fetch() or imports data directly (ALL OR NOTHING — any direct fetch = fail)
- [ ] 5.  State Management — context/reducers; no 5+ useState chains
- [ ] 6.  Read-Only / RBAC — inert attribute; role constants in domain file
- [ ] 7.  Backend Integration Markers — @backend annotations in code match domain file shapes
- [ ] 8.  Component Library Compliance — ACTIVELY SCAN for reinvented primitives; if no scan performed mark UNEVALUATED
- [ ] 9.  Design Token Compliance — no hardcoded colors, spacing, sizing, typography, or raw pixels in inline styles, CSS, or Tailwind arbitrary values
- [ ] 10. Destructive Action Safety — ConfirmDialog or Toast+Undo gates all destructive actions
- [ ] 11. Routing & Navigation — react-router-dom (or Next.js file-based); no conditional rendering for page switching
- [ ] 12. Component Performance — useMemo, useCallback, React.memo on heavy renders
- [ ] 13. Accessibility — semantic HTML, ARIA, keyboard nav, focus trapping, aria-live
- [ ] 14. Code Quality — TS: no any types, strict interfaces; JS: PropTypes on all components, JSDoc on api/domain files
- [ ] 15. Error Boundaries & Component Resilience — route-level error boundaries; PLUS every data-consuming component handles loading, error, and empty states
- [ ] 16. QA: Test Selectors & Test Existence — data-testid coverage; test files exist; test runner configured
- [ ] 17. Dependency, Environment & Onboarding — no unused deps, pinned versions, .env.example present, README has install+run instructions
- [ ] 18. File Hygiene & Icons — no orphans, unused imports, dead code; inline SVGs use library icons first
- [ ] 19. Component Production Readiness — spot-check 3-5 major components for: null/undefined handling, text overflow, empty array behavior, hardcoded content, prop interface clarity, style isolation
- [ ] 20. Security Basics — no hardcoded secrets, no dangerouslySetInnerHTML with unsanitized data, npm audit clean, .env not committed

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
# === STEP 0: STACK DETECTION ===

find src -name '*.jsx' -o -name '*.js' | head -5
find src -name '*.tsx' -o -name '*.ts' | head -5

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

# Conditional routing anti-pattern
grep -RInE 'page === |currentPage|setPage\(' src --include='*.jsx' --include='*.tsx' | head -40

# useState chains (5+ in one file)
find src \( -name '*.jsx' -o -name '*.tsx' \) | while read f; do c=$(grep -c 'useState' "$f"); if [ "$c" -ge 5 ]; then echo "$f: $c useState calls"; fi; done | sort -t: -k2 -nr | head -30

# Router library check
grep -RInE 'react-router-dom|BrowserRouter|Routes|Route|useNavigate' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | head -40

# === DIMENSION 14: CODE QUALITY ===

# any types (TS only)
grep -RInE ':\s*any\b|as any' src --include='*.tsx' --include='*.ts' | head -40

# PropTypes usage (JS — check coverage)
grep -RIl 'PropTypes' src --include='*.jsx' --include='*.js' | wc -l
find src -name '*.jsx' | wc -l

# === DIMENSION 15: RESILIENCE ===

# Error boundaries
grep -RInE 'ErrorBoundary|componentDidCatch|getDerivedStateFromError' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' | head -40

# Loading/empty state patterns
grep -RInE 'isLoading|loading|Skeleton|Spinner|emptyState|empty-state|no data|no results' src --include='*.jsx' --include='*.tsx' | head -80

# === DIMENSION 16: QA SELECTORS & TEST EXISTENCE ===

# Missing data-testid (list files without any)
find src/components \( -name '*.jsx' -o -name '*.tsx' \) | while read f; do grep -q 'data-testid' "$f" || echo "$f"; done | head -80

# Coverage count
total=$(find src/components \( -name '*.jsx' -o -name '*.tsx' \) | wc -l | tr -d ' ')
missing=$(find src/components \( -name '*.jsx' -o -name '*.tsx' \) | while read f; do grep -q 'data-testid' "$f" || echo "$f"; done | wc -l | tr -d ' ')
echo "total=$total missing=$missing"

# Test files existence
find src -name '*.test.*' -o -name '*.spec.*' | head -20
find src -name '__tests__' -type d | head -10
grep -E '"test"|"vitest"|"jest"|"playwright"|"cypress"' package.json | head -10

# === DIMENSION 17 & 18: HYGIENE ===

# Unused dependencies
npx depcheck --skip-missing 2>/dev/null

# Console statements in production code
grep -RInE 'console\.(log|warn|error)' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' --exclude='*.test.*' | head -80

# Inline SVGs
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

# === DIMENSION 20: SECURITY BASICS ===

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

# === DIMENSION 19: PRODUCTION READINESS ===

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

| Smell | Grep |
|-------|------|
| God component (400+ lines) | `find src \( -name '*.tsx' -o -name '*.jsx' \) \| xargs -I {} awk 'END{if(NR>400)print FILENAME,NR}' {}` |
| Duplicate component names | `find src -name '*.jsx' -o -name '*.tsx' \| xargs -I {} basename {} \| sort \| uniq -d` |
| Direct mock import | `grep -RInE 'MOCK_\|from.*/data/' src/components src/pages --include='*.jsx'` |
| Direct fetch in component | `grep -RInE 'fetch\(\|axios\.' src/components src/pages --include='*.jsx'` |
| Reinvented primitive | `grep -RInEi 'dropdown\|modal\|tooltip\|popover' src/components --include='*.jsx' \| grep -Evi 'radix\|shadcn\|/ui/'` |
| Hardcoded hex in Tailwind | `grep -RInE '\[#[0-9a-fA-F]{3,8}\]' src --include='*.jsx'` |
| Hardcoded spacing in Tailwind | `grep -RInE '\[[0-9.]+px\]' src --include='*.jsx'` |
| Inline style raw pixels | `grep -RInE "style=\{\{[^}]*(padding\|margin\|width\|height\|fontSize)[^}]*[0-9]+px" src --include='*.jsx'` |
| Missing loading/empty states | `grep -RIL 'isLoading\|loading\|Skeleton\|Spinner\|empty' src/components --include='*.jsx'` |
| No null guards | `grep -RInE 'props\.\w+\.' src/components --include='*.jsx' \| grep -v '?.' \| head -40` |
| Hardcoded labels | `grep -RInE "placeholder=['\"][A-Z]\|label=['\"][A-Z]" src/components --include='*.jsx'` |
| Missing @backend annotations | `grep -nE 'export\s+(async\s+)?function' src/api.js src/services/api.js 2>/dev/null \| grep -v '@backend'` |
| Missing data-testid | `find src/components -name '*.jsx' \| while read f; do grep -q 'data-testid' "\$f" \|\| echo "\$f"; done \| head -40` |
| Interaction: setTimeout visuals | `grep -RInE 'setTimeout\|setInterval' src/components --include='*.jsx'` |
| Fixed widths preventing responsive | `grep -RInE 'width:\s*[0-9]{3,}px\|w-\[[0-9]{3,}px\]' src/components --include='*.jsx'` |
| Hardcoded API keys | `grep -RInE 'sk_live\|pk_live\|AKIA\|api_key\s*[:=]' src --include='*.jsx' --include='*.js'` |
| dangerouslySetInnerHTML | `grep -RIn 'dangerouslySetInnerHTML' src --include='*.jsx' --include='*.tsx'` |
| No test files | `find src -name '*.test.*' -o -name '*.spec.*' \| wc -l` |
| README missing setup | `grep -qiE 'npm install\|npm run dev' README.md \|\| echo 'MISSING'` |
| Circular imports | Check if file A imports B and B imports A |
| Relative deep imports | `grep -RInE '\.\./\.\.' src --include='*.jsx' --include='*.tsx' \| head -20` |

---

## Target file layout

See SKILL.md "Target File Layout" section — single source of truth, not duplicated here to avoid drift.
