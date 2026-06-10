# Mode: Refactor

> Load this file when the user runs `/vibe-to-prod-v2 refactor` or gives no explicit mode. This is the default mode — it modifies files.

---

## Refactor execution order

Work through dimensions in this order. It minimizes regression risk — architecture changes first, then logic, then surface-level cleanup.

1. **Dimension 19 spot-check first** — identify components that would crash with real data before touching anything. These are the highest risk areas.
2. **Dimension 2** — extract mock data to `src/data/` before restructuring anything else
3. **Dimension 3** — establish domain types / JSDoc typedefs before writing any stubs
4. **Dimension 4** — create API stubs + data fetching hooks
5. **Dimension 7** — add `// @backend` annotations to every stub
6. **Dimension 1** — split god-components (types and data must be stable first)
7. **Dimension 1b** — consolidate duplicated primitives
8. **Dimension 8** — replace reinvented UI primitives
9. **Dimension 5** — consolidate useState chains into reducers/stores
10. **Dimension 6** — add role constants and inert-based read-only guards
11. **Dimension 9** — replace hardcoded visual values with design tokens
12. **Dimension 10** — add ConfirmDialog to destructive actions
13. **Dimension 11** — introduce routing (or Next.js file structure)
14. **Dimension 12** — add memoization where measurable
15. **Dimension 13** — accessibility fixes
16. **Dimension 14** — code quality (types, PropTypes, linting)
17. **Dimension 15** — error boundaries + component resilience states
18. **Dimension 16** — add data-testid selectors
19. **Dimension 17** — dependency and environment hygiene
20. **Dimension 18** — file hygiene and icon replacement

---

## Non-negotiable rules during refactor

- **Never alter the design surface.** Layout, spacing, animations, transitions — untouched. If a refactor would visually change anything, stop and flag it instead.
- **Never introduce `any` types** in TypeScript as a shortcut.
- **Never add `// @ts-ignore`** to suppress errors.
- **Never introduce new features.** Refactor only — no new functionality.
- **DOM Hierarchy Guard:** when splitting components, the rendered HTML output must be byte-for-byte identical in structure. Verify before and after.

---

## Quick mode

If the user runs `/vibe-to-prod-v2 quick`, apply only these dimensions:
1, 2, 3, 4, 7, 17, 18

Skip all others. State at the start: "Running quick mode — covering architecture, data extraction, types, API stubs, backend markers, and hygiene only."

---

## After refactoring

Tell the user:

> "Refactor complete. The `// @backend` annotations in `src/api/index.ts` are the developer's integration guide — every endpoint they need to build is documented there inline.
>
> Before handing off, run the app and verify everything looks and feels identical to before. Then run `/vibe-to-prod-v2 audit` for a final check."
