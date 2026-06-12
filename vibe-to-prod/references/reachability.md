# Reachability Pass — Deterministic Orphan Detection

> **Run this once at the start of every audit and at the start of fix-it-all (Path 1 step 2).** It replaces grep-by-hand orphan detection. Hand-grepping is unreliable — across test runs the same codebase returned 8, 24, and 57 orphans depending on which files the agent thought to check. A real import-graph traversal returns the complete, identical list every time, on every model.

## Why a script, not grep

Grep counts text matches; it cannot follow the import graph. A file can look orphaned (no one greps its name) while being reached via a barrel export, or look used (its name appears) while only living in a comment. The only reliable method is to start at the entry point and walk every import edge, including `export ... from`, dynamic `import()`, and `React.lazy`.

## The script

Write this to a temp file and run with `node`. It starts at `src/main.tsx` (adjust if the entry differs), resolves `@/` aliases, relative paths, index files, and all four extensions, then walks the full graph including lazy/dynamic imports.

```js
const fs = require("fs");
const path = require("path");
const root = process.cwd();
const srcRoot = path.join(root, "src");
const entry = path.join(srcRoot, "main.tsx"); // adjust if entry differs (index.tsx, etc.)
const exts = [".ts", ".tsx", ".js", ".jsx"];

function read(file) {
  try {
    return fs.readFileSync(file, "utf8");
  } catch {
    return null;
  }
}

function resolveImport(fromFile, spec) {
  let base;
  if (spec.startsWith("@/")) base = path.join(srcRoot, spec.slice(2));
  else if (spec.startsWith("."))
    base = path.resolve(path.dirname(fromFile), spec);
  else return null; // bare package import — not a local file
  const candidates = [];
  for (const ext of exts) candidates.push(base + ext);
  for (const ext of exts) candidates.push(path.join(base, "index" + ext));
  candidates.push(base);
  return candidates.find((f) => fs.existsSync(f)) || null;
}

const reachable = new Set();
const queue = [entry];
while (queue.length) {
  const file = queue.pop();
  if (!file || reachable.has(file)) continue;
  reachable.add(file);
  const text = read(file);
  if (!text) continue;
  // static imports, re-exports, AND dynamic/lazy imports — all three matter
  const patterns = [
    /import\s+(?:[^'";]+\s+from\s+)?['"]([^'"]+)['"]/g,
    /export\s+[^'";]+\s+from\s+['"]([^'"]+)['"]/g,
    /import\(\s*['"]([^'"]+)['"]\s*\)/g,
  ];
  for (const regex of patterns) {
    for (const match of text.matchAll(regex)) {
      const resolved = resolveImport(file, match[1]);
      if (resolved && resolved.startsWith(srcRoot) && !reachable.has(resolved))
        queue.push(resolved);
    }
  }
}

// Inventory every component/source file, then subtract the reachable set
const all = [];
function walk(dir) {
  for (const entry of fs.readdirSync(dir, { withFileTypes: true })) {
    const full = path.join(dir, entry.name);
    if (entry.isDirectory()) walk(full);
    else if (entry.isFile() && /\.(tsx|ts)$/.test(entry.name)) all.push(full);
  }
}
walk(srcRoot);

const orphans = all
  .filter((f) => !reachable.has(f))
  .map((f) => path.relative(root, f))
  .sort();

// Two-bucket split: ui/ library primitives vs app/variant orphans
const uiOrphans = orphans.filter((f) => /\/ui\//.test(f));
const appOrphans = orphans.filter((f) => !/\/ui\//.test(f));

console.log(
  `reachable=${all.length - orphans.length}  orphans=${orphans.length}`,
);
console.log(
  "\n--- APP / VARIANT ORPHANS (skip for refactoring; safe to flag for deletion) ---",
);
appOrphans.forEach((f) => console.log(f));
console.log(
  "\n--- UI LIBRARY PRIMITIVES (unused, but DO NOT delete — they are a component library) ---",
);
uiOrphans.forEach((f) => console.log(f));
```

## The two-bucket distinction — critical

The script splits orphans into two groups, and the audit MUST treat them differently:

- **App / variant orphans** (everything outside `ui/`): these are unused screen variants, dead exploration components, and Figma Make export artifacts. The designer vibecoded many versions and wired up some. These are skip-for-refactoring AND safe to flag for deletion. List them in the audit's "Orphaned files" section.

- **ui/ library primitives** (e.g. `carousel.tsx`, `calendar.tsx`, `drawer.tsx`): shadcn/Radix components that ship with the kit but aren't used yet. These are NOT dead exploration — they are a component library the designer may pull from next week. **Never flag these for deletion and never refactor them.** Note them separately as "unused library primitives — leave in place." Deleting `card.tsx` because it's currently unused would be wrong.

## Using the output

- **Audit:** put the app/variant orphans in the dedicated "Orphaned files (skip for refactoring)" report section. Note ui/ primitives separately, neutrally.
- **Fix-it-all:** mark every app/variant orphan `SKIP — orphaned` in the progress ledger before the first edit. Never split, type-fix, icon-swap, or touch an orphaned file — not in round one, not in round two, regardless of size. A 2,000-line orphaned god-component is skipped entirely.

## If the script fails

If `node` errors (unusual entry path, non-standard structure), fall back to per-file grep but state in the report that the reachability pass was incomplete and the orphan list may miss files. Don't silently revert to the unreliable method.
