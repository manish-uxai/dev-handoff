# Dimensions: Architecture
# Covers: 1 (Component Architecture), 2 (Data Extraction), 11 (Routing)

---

## Dimension 1: Component Architecture & DOM Hierarchy Guard

### God-components
Break any component over 400 lines into focused child modules.
- Separate stateful logic from presentational components
- One domain context per provider
- **DOM Hierarchy Guard:** When splitting, preserve the exact DOM layout hierarchy. No redundant wrapper elements. No altered CSS display properties (`flex`, `grid`). A split that shifts layout by even 1px is a failed split.

### Absolute path aliases
Enforce `@/*` mapping. No `../../` relative imports anywhere in the codebase.
- Configure in `tsconfig.json` (TS) or `jsconfig.json` (JS)
- In audit mode, flag every import using `../..` as Low severity

### Dimension 1b: Reusability pass (separate audit step)
Scan the entire codebase — not just the audited module — for duplicated UI patterns.

**Detection methods (run all four):**

1. **Duplicate filenames across folders**
```bash
find src -name '*.jsx' -o -name '*.tsx' | xargs -I {} basename {} | sort | uniq -d
```

2. **Overlapping className patterns**
```bash
grep -RhoE 'className="[^"]{60,}"' src --include='*.jsx' --include='*.tsx' | sort | uniq -d | head -20
```

3. **Copy-pasted JSX structures** — read the `components/` directory and flag files with overlapping element trees or near-identical prop patterns

4. **Directory structure review** — list all files in `components/` and flag any that serve the same purpose (e.g., `UserCard.tsx` and `ProfileTile.tsx` both rendering a user name + avatar)

In audit mode, list every duplicated pattern found with file locations. Mark as Medium severity. This step is not optional — if not performed, mark dimension 1b as INCOMPLETE.

---

## Dimension 2: Clean Data Extraction

- Move all `MOCK_` arrays, seed data, and hardcoded lists to `src/data/`
- No inline arrays in UI components or pages
- Preserve the exact data shape when extracting — changing shape causes rendering errors
- `src/data/` files are consumed only by `src/api/index.ts` — never imported directly by components

---

## Dimension 11: Routing & Navigation

> If stack is Next.js, this dimension is overridden by `stacks/nextjs.md`. Skip this and apply the Next.js routing rules instead.

**For React with react-router-dom:**
- Replace all conditional page rendering (`if (page === 'home')`, `currentView === 'dashboard'`) with declarative `react-router-dom` paths
- Implement lazy loading (`React.lazy`) for heavy route-level containers
- Set up Route Guards (`<ProtectedRoute />`) for authenticated/role-based views

Flag conditional routing as High severity — it prevents a developer from understanding the app structure without reading root component logic.

---

## In audit mode

For each dimension, show grep evidence before stating findings. Do not state conclusions without file + line citations.

Dimension 1b must appear as a separate row in the audit output. It cannot be merged into dimension 1.
