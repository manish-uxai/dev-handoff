# Dimensions: State & Access
# Covers: 5 (State Management), 6 (Read-Only / RBAC)

---

## Dimension 5: State Management & Reducer Isolation

**useState chain smell:** 5 or more independent `useState` hooks in a single component causes cascading re-renders and makes the component hard to maintain. Consolidate into a reducer or context.

**Detection:**
```bash
find src \( -name '*.jsx' -o -name '*.tsx' \) | while read f; do
  c=$(grep -c 'useState' "$f")
  if [ "$c" -ge 5 ]; then echo "$f: $c useState calls"; fi
done | sort -t: -k2 -nr
```

**Fix:** consolidate into a reducer-based state slice with explicit action schemas. This allows a developer to hook up Redux, Zustand, or any other state manager by replacing only the reducer, not every component.

**Library detection:**
- If Zustand already in `package.json` — consolidate into a Zustand store in `src/store/`
- If Redux present — use Redux slice pattern
- If nothing present — use `useReducer` + Context, or recommend Zustand: `npm install zustand`

**Next.js addition:** flag any `useState`, `useContext`, or `useReducer` inside a Server Component as High severity. State hooks only work in Client Components. Files using them need `"use client"` at the top.

---

## Dimension 6: Read-Only / Role-Based Access Control

**Blocking UI for read-only roles:**
Use the HTML `inert` attribute on interactive containers. This blocks all interaction without altering styles.

```tsx
<form inert={isReadOnly ? true : undefined}>
  {/* all inputs, buttons frozen for read-only users */}
</form>
```

Do not use CSS `pointer-events: none` alone — it only blocks mouse events, not keyboard navigation.

**Role definitions:**

TypeScript:
```ts
// in domain.ts
export type UserRole = 'admin' | 'editor' | 'viewer';
```

JavaScript:
```js
// in domain.js
export const ROLES = {
  ADMIN: 'admin',
  EDITOR: 'editor',
  VIEWER: 'viewer',
};
```

**In audit mode:**
```bash
# Check for inert usage
grep -RIn 'inert=' src --include='*.jsx' --include='*.tsx' | head -20

# Check for role definitions
grep -RInE 'UserRole|ROLES\s*=|viewer|admin|readOnly' src/types src/context src/domain --include='*.ts' --include='*.js' | head -20
```

If the app has no role-based access requirements, note it and mark this dimension as not applicable.
