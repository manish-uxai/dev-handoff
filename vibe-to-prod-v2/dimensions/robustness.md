# Dimensions: Robustness & Production Readiness
# Covers: 13 (Accessibility), 14 (Code Quality), 15 (Error Boundaries), 19 (Component Production Readiness)

---

## Dimension 13: Accessibility Baseline

- Eliminate "div soup" — use semantic HTML5 landmarks (`<nav>`, `<main>`, `<header>`, `<button>`, `<aside>`)
- Interactive elements need `aria-label` or `aria-labelledby`
- Keyboard focus: proper `tabIndex`, visible focus states, focus trapping in modal dialogs
- `aria-live` regions for dynamic real-time content updates

Note: replacing hand-rolled primitives with shadcn/Radix automatically resolves most a11y gaps for those components.

```bash
grep -RInE '<main|<nav|<header|<aside|aria-label|aria-labelledby|aria-live|role=' \
  src --include='*.jsx' --include='*.tsx' | head -80
```

---

## Dimension 14: Code Quality

Apply the correct path based on detected stack (see `stacks/react.md`).

**TypeScript:** no `any` types, strict interfaces, organized imports, ESLint/Prettier compliance

**JavaScript:** PropTypes on all components, JSDoc on all `api/index.js` and `domain.js` functions, organized imports, ESLint/Prettier compliance

**PropTypes coverage check:**
```bash
# How many JSX files have PropTypes vs total
grep -RIl 'PropTypes' src --include='*.jsx' --include='*.js' | wc -l
find src -name '*.jsx' | wc -l
```

Flag low PropTypes coverage as Medium severity — a developer can't understand component contracts without them.

---

## Dimension 15: Error Boundaries & Component Resilience

### Route-level
- Wrap major route views in React Error Boundaries (or Next.js `error.tsx` files)
- Global Toast/Snackbar channel for async API failures — errors should surface to the user, not disappear silently

### Component-level (every data-consuming component)
Every component that receives async data must handle three states:

| State | What to show |
| :--- | :--- |
| **Loading** | Skeleton, spinner, or placeholder — never blank space |
| **Error** | Fallback message — never a crash |
| **Empty** | Helpful message ("No patients found") — never an invisible component |

```bash
grep -RInE 'isLoading|loading|Skeleton|Spinner|emptyState|empty-state|"no data"|"no results"' \
  src --include='*.jsx' --include='*.tsx' | head -60
```

---

## Dimension 19: Component Production Readiness

This dimension checks whether components survive real-world data conditions, not just the prototype's happy-path mock data.

### Spot-check requirement
In audit mode, pick 3-5 major data-consuming components and check each one. Report findings per component. Do not skip this step.

### Data resilience checks (per component)

**Null/undefined fields:**
Does the component crash when a data field is missing?
```bash
# Components accessing props without null guards
grep -RInE 'props\.\w+\.' src/components --include='*.jsx' --include='*.tsx' | \
  grep -v '?.' | grep -v '&&' | head -40
```
Flag any component that would crash on `patient.address.city` if `address` is undefined.

**Text overflow:**
What happens with a 200-character name, a long URL, or a multi-paragraph description?
- Look for fixed-width containers with no `truncate`, `overflow-hidden`, or `line-clamp`
- Long strings that would break layout = Medium severity

**Variable data lengths:**
Does the component work with 0 items, 1 item, 500 items?
- 0 items → empty state should show (covered by dimension 15)
- 500 items → check for virtualization or pagination if rendering a list

**Special characters:**
Quotes, ampersands, unicode, emoji — do they render or cause encoding issues? Usually fine in React but worth noting if you see concatenated strings going into `dangerouslySetInnerHTML`.

### Reusability checks

**Hardcoded content:**
Labels, placeholder text, error messages, and URLs baked directly into a component instead of passed as props.
```bash
grep -RInE "placeholder=['\"][A-Z]|label=['\"][A-Z]" src/components --include='*.jsx' --include='*.tsx' | head -40
```
Flag as Low severity — reduces reusability but won't break production.

**Style isolation:**
Does the component depend on a specific parent's CSS for its own layout to work? Test by looking for components that use `position: absolute` or rely on a parent's `position: relative` without setting it themselves.

**Prop interface clarity:**
A developer should understand what a component accepts without reading its internals. PropTypes (JS) or a `Props` interface (TS) is the minimum. Flag missing prop documentation as Medium severity.

### Responsive basics
- No fixed pixel widths that prevent the component from adapting

```bash
grep -RInE 'width:\s*[0-9]{3,}px|w-\[[0-9]{3,}px\]|minWidth:\s*[0-9]{3,}' \
  src/components --include='*.jsx' --include='*.tsx' | head -40
```

If responsive behavior is explicitly out of scope for the project, note it once and move on.
