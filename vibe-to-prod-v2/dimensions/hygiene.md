# Dimensions: Hygiene
# Covers: 16 (QA Selectors), 17 (Dependencies & Environment), 18 (File Hygiene & Icons)

---

## Dimension 16: QA & Test Selectors

Add `data-testid` attributes to all primary interactive elements, form controls, and major layout sections.

This allows QA engineers and developers to write automated tests without modifying production markup.

```bash
# Files missing data-testid entirely
find src/components \( -name '*.jsx' -o -name '*.tsx' \) | \
  while read f; do grep -q 'data-testid' "$f" || echo "$f"; done | head -60

# Coverage count
total=$(find src/components \( -name '*.jsx' -o -name '*.tsx' \) | wc -l | tr -d ' ')
missing=$(find src/components \( -name '*.jsx' -o -name '*.tsx' \) | \
  while read f; do grep -q 'data-testid' "$f" || echo "$f"; done | wc -l | tr -d ' ')
echo "total=$total missing=$missing"
```

Flag high miss rate as High severity — QA cannot automate without these.

---

## Dimension 17: Dependency & Environment Hygiene

**Unused dependencies:**
```bash
npx depcheck --skip-missing 2>/dev/null
```

Remove unused packages. They increase bundle size and create security exposure.

**Version pinning:**
```bash
node -e "
const p = require('./package.json');
const deps = {...p.dependencies, ...p.devDependencies};
const loose = Object.entries(deps).filter(([k,v]) => /^\^|~/.test(v));
console.log('loose versions: ' + loose.length);
console.log(loose.slice(0,10).map(([k,v]) => k+': '+v).join('\n'));
"
```

Pin critical dependency versions (React, react-dom, routing, state management). Loose `^` versions on core packages can cause breaking changes on `npm install`.

**Environment variables:**
```bash
ls -la .env.example 2>/dev/null || echo '.env.example missing'
```

`.env.example` must exist with all required variables stubbed. No real values — only placeholder names and descriptions.

---

## Dimension 18: File Hygiene & Icon Consolidation

**Dead code cleanup:**
- Delete orphaned files (imported nowhere)
- Remove unused imports
- Remove `console.log` / `console.error` from production code

```bash
# Console statements in production code
grep -RInE 'console\.(log|warn|error)' src \
  --include='*.jsx' --include='*.tsx' --include='*.js' --include='*.ts' \
  --exclude='*.test.*' | head -60
```

**Icon replacement (lookup first, extract last):**

For every inline SVG found in a component:

1. Identify what the icon represents — use file name, component name, and surrounding code as context
2. Check **lucide-react** first — 1500+ icons, covers most prototype use cases
3. If not found, check **heroicons**, then **phosphor-icons**
4. If a library equivalent exists — replace the entire SVG block with a one-line import
5. Only extract to `components/icons/` if no library equivalent exists

```bash
# Find components with inline SVGs
grep -RIl '<svg' src/components src/pages --include='*.jsx' --include='*.tsx' | head -40
```

Never leave raw multi-line SVG coordinate paths (`<path d="M 12 3 L 4.5..."/>`) inside UI components. They clutter JSX and make it unreadable.
