# Mode: Audit

> Load this file when the user runs `/vibe-to-prod-v2 audit`. Report violations without modifying any files.

---

## Audit execution order

1. State which files you loaded (SKILL.md + which stack + which dimensions)
2. Run the full grep evidence pack
3. Spot-check 3-5 components for dimension 19
4. Score all dimensions
5. Output the report in the format below

---

## Required grep evidence pack

Run all of these before writing a single finding. Include output summaries in the report.

```bash
# === STACK DETECTION ===
find src -name '*.jsx' -o -name '*.js' | head -5
find src -name '*.tsx' -o -name '*.ts' | head -5

# === DIMENSION 1: ARCHITECTURE ===
find src \( -name '*.tsx' -o -name '*.jsx' \) | xargs -I {} awk 'END{if(NR>400)print FILENAME, NR}' {}
grep -RhoE 'className="[^"]{60,}"' src --include='*.jsx' --include='*.tsx' | sort | uniq -d | head -20
find src \( -name '*.jsx' -o -name '*.tsx' \) | xargs -I {} basename {} | sort | uniq -d
grep -RInE '\.\./\.\.' src --include='*.jsx' --include='*.tsx' --include='*.ts' --include='*.js' | head -20

# === DIMENSION 2 & 4: DATA FLOW ===
grep -RInE 'MOCK_|from.*/data/' src/components src/pages --include='*.jsx' --include='*.tsx' --include='*.ts' | head -40
grep -RInE 'fetch\(|axios\.|\.get\(|\.post\(' src/components src/pages --include='*.jsx' --include='*.tsx' | head -40

# === DIMENSION 7: BACKEND MARKERS ===
grep -nE 'export\s+(async\s+)?function' src/api/index.ts src/api/index.js src/services/api.ts src/services/api.js 2>/dev/null | grep -v '@backend'

# === DIMENSION 8: COMPONENT LIBRARY ===
grep -RInEi 'dropdown|modal|tooltip|popover|datepicker|date-picker|tablist|select.*option' \
  src/components --include='*.jsx' --include='*.tsx' | grep -Evi 'node_modules|shadcn|radix|headlessui|/ui/' | head -60

# === DIMENSION 9: DESIGN TOKENS ===
grep -RInE '#[0-9a-fA-F]{3,8}|rgb\(|hsl\(' src --include='*.jsx' --include='*.tsx' --include='*.css' --include='*.scss' | wc -l
grep -RInE '\[#[0-9a-fA-F]{3,8}\]' src --include='*.jsx' --include='*.tsx' | head -20
grep -RInE '\[[0-9.]+px\]|\[[0-9.]+rem\]' src --include='*.jsx' --include='*.tsx' | head -20
grep -RInE "style=\{\{[^}]*(padding|margin|width|height|fontSize)[^}]*[0-9]+px" src --include='*.jsx' --include='*.tsx' | head -20

# === DIMENSION 5 & 11: STATE & ROUTING ===
grep -RInE 'page === |currentPage|setPage\(' src --include='*.jsx' --include='*.tsx' | head -20
grep -RInE 'react-router-dom|BrowserRouter|Routes|Route' src --include='*.jsx' --include='*.tsx' | head -20
find src \( -name '*.jsx' -o -name '*.tsx' \) | while read f; do c=$(grep -c 'useState' "$f"); if [ "$c" -ge 5 ]; then echo "$f: $c"; fi; done | sort -t: -k2 -nr | head -20

# === DIMENSION 14: CODE QUALITY ===
grep -RInE ':\s*any\b|as any' src --include='*.tsx' --include='*.ts' | head -20
grep -RIl 'PropTypes' src --include='*.jsx' --include='*.js' | wc -l

# === DIMENSION 15: RESILIENCE ===
grep -RInE 'ErrorBoundary|componentDidCatch' src --include='*.jsx' --include='*.tsx' | head -20
grep -RInE 'isLoading|loading|Skeleton|Spinner' src --include='*.jsx' --include='*.tsx' | wc -l

# === DIMENSION 16: QA SELECTORS ===
total=$(find src/components \( -name '*.jsx' -o -name '*.tsx' \) | wc -l | tr -d ' ')
missing=$(find src/components \( -name '*.jsx' -o -name '*.tsx' \) | while read f; do grep -q 'data-testid' "$f" || echo "$f"; done | wc -l | tr -d ' ')
echo "data-testid: total=$total missing=$missing"

# === DIMENSION 17: HYGIENE ===
ls -la .env.example 2>/dev/null || echo '.env.example missing'
npx depcheck --skip-missing 2>/dev/null

# === DIMENSION 18: ICONS & FILE HYGIENE ===
grep -RIl '<svg' src/components src/pages --include='*.jsx' --include='*.tsx' | head -20
grep -RInE 'console\.(log|warn|error)' src --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' --exclude='*.test.*' | wc -l

# === DIMENSION 19: PRODUCTION READINESS ===
grep -RInE 'props\.\w+\.' src/components --include='*.jsx' --include='*.tsx' | grep -v '?.' | grep -v '&&' | head -20
grep -RInE "placeholder=['\"][A-Z]|label=['\"][A-Z]" src/components --include='*.jsx' --include='*.tsx' | head -20
grep -RInE 'width:\s*[0-9]{3,}px|w-\[[0-9]{3,}px\]' src/components --include='*.jsx' --include='*.tsx' | head -20

# === FLAGGED INTERACTIONS ===
grep -RInE 'stiffness:\s*[5-9][0-9]{2,}|damping:\s*[0-4]\b' src --include='*.jsx' --include='*.tsx' | head -20
grep -RInE 'transition.*0ms|transition.*0s[^0-9]|duration-0' src --include='*.jsx' --include='*.tsx' --include='*.css' | head -20
grep -RInE 'setTimeout|setInterval' src/components src/pages --include='*.jsx' --include='*.tsx' | head -40
grep -RInE 'onMouse|onHover|onFocus.*set[A-Z]' src/components --include='*.jsx' --include='*.tsx' | head -20
```

---

## Audit report format

```
## Audit: [directory or file]
**Stack:** [React/TS | React/JS | Next.js]
**Files loaded:** [list of files read]

Visual fidelity should be verified in the browser after any refactoring.

---

### Flagged Interactions
[List findings with file + line, or state "No interaction smells found."]

---

### Dimension Results

| # | Dimension | Status | Severity | Evidence |
|---|-----------|--------|----------|----------|
| 1 | Component Architecture | ✗ Fail | High | [file:line — specific finding] |
| 1b | Reusability Pass | ✗ Fail | Medium | [file:line — specific finding] |
| 2 | Data Extraction | ✓ Pass | — | [file:line — evidence of passing] |
...

---

### Recommended Fix Order
1. [Highest developer impact first — not code purity first]
2. ...

---

### Dimensions Unevaluated
[List any that could not be evaluated with reason]
```

---

## Audit rules

1. **Evidence before conclusions.** Run grep pack first. Write findings second. Never the reverse.

2. **Distinct citations only.** Each citation must be a distinct file + line. Citing `domain.js` four times as four separate findings is padding. Cite `domain.js:45`, `domain.js:112` if they're genuinely different issues.

3. **Dimension 4 is all-or-nothing.** One direct fetch anywhere = fail. No partial pass.

4. **Dimension 8 must be actively scanned.** If grep was not run, mark UNEVALUATED.

5. **Dimension 1b is a separate row.** Never merge into dimension 1.

6. **Dimension 19 requires named spot-checks.** List the 3-5 components checked and what was found per component.

7. **Every severity needs a reason.** "High" alone is not acceptable. One sentence explaining why.

8. **Flagged interactions section is required.** Either list findings or explicitly state none found. Never skip.

9. **"Dimensions Unevaluated: None" must appear** at the end confirming all 19 were assessed.
