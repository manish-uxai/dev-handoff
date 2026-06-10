# Dimensions: UI Quality
# Covers: 8 (Component Library), 9 (Design Tokens), 10 (Destructive Actions), 12 (Performance)

---

## Dimension 8: Component Library Compliance

**Actively scan** for reinvented UI primitives. Do not passively assume compliance. If no scan was performed, mark as UNEVALUATED — not passing.

**What to look for:**
```bash
grep -RInEi 'dropdown|modal|tooltip|popover|datepicker|date-picker|tablist|select.*option' \
  src/components --include='*.jsx' --include='*.tsx' | \
  grep -Evi 'node_modules|shadcn|radix|headlessui|/ui/' | head -80
```

**Two types of custom components:**

| Type | Definition | Action |
| :--- | :--- | :--- |
| Reinvented primitive | Hand-rolled dropdown, modal, tooltip, tabs, date picker | Replace with shadcn/ui or Radix UI equivalent |
| Genuine custom | Novel interaction, product-specific visualization | Preserve and harden |

**Replacing primitives:**
- Preserve the designer's exact visual styling — only swap the underlying machinery
- shadcn/ui and Radix UI components are production-tested, accessible, and ship without needing a design system
- A developer can use them as-is or swap later if they have their own component library

**Consolidating duplicates:**
Same button, card, badge, or input built with slight variations across files → one shared primitive in `components/ui/`. A developer should be able to find and understand every UI primitive in one place.

---

## Dimension 9: Design Token Compliance

Covers **all visual values** — not just colors.

**What to scan:**

```bash
# Hardcoded hex colors
grep -RInE '#[0-9a-fA-F]{3,8}|rgb\(|hsl\(' src --include='*.jsx' --include='*.tsx' --include='*.css' --include='*.scss' | head -60

# Tailwind arbitrary color values
grep -RInE '\[#[0-9a-fA-F]{3,8}\]' src --include='*.jsx' --include='*.tsx' | head -40

# Tailwind arbitrary spacing/sizing
grep -RInE '\[[0-9.]+px\]|\[[0-9.]+rem\]|\[[0-9.]+em\]' src --include='*.jsx' --include='*.tsx' | head -60

# Inline style raw pixels
grep -RInE "style=\{\{[^}]*(padding|margin|width|height|fontSize|gap|borderRadius|lineHeight)[^}]*[0-9]+px" \
  src --include='*.jsx' --include='*.tsx' | head -60

# CSS raw pixel values
grep -RInE '(padding|margin|width|height|font-size|gap|border-radius|line-height):\s*[0-9]+px' \
  src --include='*.css' --include='*.scss' | head -60
```

**Fix approach:**
- Map colors to semantic CSS variables: `--color-status-danger`, `--color-brand-primary`
- Map spacing/sizing to Tailwind scale or CSS variables
- **Defensive snapping:** only snap Tailwind arbitrary values to the nearest standard scale if visual shift < 1px. Otherwise extract to a CSS variable.

---

## Dimension 10: Destructive Action Safety

Gate all deletions, resets, and irreversible state changes behind a confirmation step.

Two acceptable patterns:
- `ConfirmDialog` — modal asking "Are you sure?" before executing
- "Toast with Undo" — execute immediately, show a toast with an undo option for a few seconds

No immediate destructive actions without one of these patterns. Flag any `onClick` that directly calls delete/remove/reset without a guard as Medium severity.

```bash
grep -RInE 'onClick=\{.*delete|\.remove\(|\.reset\(' src --include='*.jsx' --include='*.tsx' | head -60
```

---

## Dimension 12: Component Performance

Apply these where they make a measurable difference — not everywhere by default.

- `useMemo` — expensive client-side filtering, sorting, or calculations that run on every render
- `useCallback` — event handler functions passed down to child components that would otherwise cause unnecessary re-renders
- `React.memo` — heavy list items, cards, or data grid rows rendered in large lists

Do not apply memoization to simple components or trivial values. Over-memoization adds complexity without benefit.
