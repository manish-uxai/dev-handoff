# HTML to React Conversion Guide

> **Read this when Step 0 detects a plain HTML/CSS project (no package.json, no React).** The designer's visual work has value — the goal is to preserve their design while converting the technology to Vite + React + TypeScript + shadcn/Radix. The conversion changes the stack, not the design.
>
> This guide is also used when a designer drops Figma MCP HTML output or agent-generated HTML into the workspace.

---

## Pre-flight

Before converting:

- [ ] Run `node --version` — if missing, stop and direct designer to install (see SKILL.md pre-flight)
- [ ] Inventory what's there: how many HTML files, any CSS files, any JavaScript, any images/assets
- [ ] Note whether CSS is inline, in a `<style>` tag, or in external stylesheets
- [ ] Note any existing Tailwind classes (Figma MCP sometimes outputs these)
- [ ] Check for vanilla JS: event listeners, DOM manipulation, fetch calls

---

## Step 1: Scaffold the React project

Run the scaffold (Path A from `references/scaffold.md`): Vite + React + TypeScript + shadcn + Tailwind.

This creates the empty production-ready structure. The HTML files stay untouched at this point — they're the reference, not the output.

---

## Step 2: Extract the design system into design.md

Before converting any markup, extract the visual rules from the HTML into a `design.md` at the project root. This is the designer's source of truth — human-readable, in design language, structured so an agent can build new on-brand pages without seeing the existing code.

**Use `references/design-md-template.md` as the structure.** Fill each section from what actually exists in the HTML — don't invent tokens that aren't represented. The template's sections, in order: Overview, Colors, Typography, Layout, Elevation & Depth, Shapes, Components, Do's and Don'ts, Responsive behavior, Known gaps.

Key extraction targets when scanning the HTML/CSS:

- **Colors:** every hex/rgb/hsl, grouped by role (brand, surface, text, semantic), each with a usage note
- **Typography:** font families (display/body/mono), sizes, weights, line-heights → the hierarchy table
- **Spacing:** repeated padding/margin values → a consistent scale
- **Components:** card styles, button variants, input styles → component specs with token references

Use `{token.name}` syntax throughout — this is the bridge to the theme CSS. Components reference tokens, never raw hex. If the HTML only supports a partial design.md (e.g. no responsive info), write what's there and note the gap.

---

## Step 3: Generate the theme from design.md

Read `design.md` and write the corresponding CSS tokens to the theme file (typically `src/index.css` or `src/styles/theme.css`):

```css
:root {
  --primary: #6366f1;
  --background: #ffffff;
  --card: #f8fafc;
  --border: #e2e8f0;
  --foreground: #0f172a;
  --muted-foreground: #64748b;
  --destructive: #ef4444;
  --success: #22c55e;
  --radius: 8px;
  --radius-sm: 6px;
}
```

This is the system implementation of design.md. design.md is the source of truth — if a designer updates design.md, the theme should be regenerated to match.

---

## Step 4: Map HTML patterns to shadcn components

Before converting line by line, scan the HTML for patterns that have shadcn equivalents. Convert directly to shadcn — don't create plain HTML React components that later get replaced in the dimension 8 pass. One conversion, not two.

| HTML pattern                  | shadcn component                                                         |
| ----------------------------- | ------------------------------------------------------------------------ |
| `<button>` with any styling   | `Button` (pick variant: default, secondary, outline, ghost, destructive) |
| Custom dropdown/select markup | `DropdownMenu` or `Select`                                               |
| Modal/popup/overlay           | `Dialog`                                                                 |
| Form inputs                   | `Input`, `Textarea`, `Select`, `Switch`, `Checkbox`                      |
| Card-like containers          | `Card` (with `CardHeader`, `CardContent`, `CardFooter`)                  |
| Data tables                   | `Table` (with `TableHeader`, `TableRow`, `TableCell`)                    |
| Tooltip on hover              | `Tooltip`                                                                |
| Tab navigation                | `Tabs` (with `TabsList`, `TabsTrigger`, `TabsContent`)                   |
| Toast/notification            | `Sonner` or `Toast`                                                      |
| Badges/tags/chips             | `Badge`                                                                  |
| Alert/info boxes              | `Alert`                                                                  |
| Navigation links              | `NavigationMenu` or plain `Link` from react-router                       |
| Sidebar layout                | `Sidebar` if shadcn has one, otherwise custom with tokens                |

**Icons:** scan for inline SVGs or icon images. Check lucide-react first for equivalents. Only create custom icon components if no match exists.

---

## Step 5: Convert pages to components

For each HTML page or major section:

**Structure:**

- The page's `<body>` content becomes the component's JSX return
- Each visually distinct section (header, sidebar, main content, footer) becomes its own component
- Repeated elements (cards in a grid, rows in a list) become mapped components

**Styles:**

- Inline `style="color: #6366f1"` → `className="text-[var(--primary)]"` using the extracted tokens
- Inline `style="padding: 16px"` → `className="p-4"` (snap to Tailwind scale)
- CSS classes from external stylesheets → convert to Tailwind utilities with tokens
- If Tailwind can't express a property, use a CSS module — never lose the visual

**Static content:**

- Text that's structural (headings, labels) → keep inline initially, extract to props later if the component will be reused
- Data that's clearly mock (lists of items, table rows) → move to `src/data/` and import via API stubs
- Images → move to `public/` folder, reference with correct paths

**JavaScript:**

- `document.getElementById('x').addEventListener('click', ...)` → React `onClick` handler with `useState`
- `element.classList.toggle('hidden')` → conditional rendering with state: `{isOpen && <Modal />}`
- `fetch('https://...')` → move to `api.ts` as a self-documenting async stub (descriptive name + typed return)
- DOM manipulation (`innerHTML`, `appendChild`) → React state driving JSX

**Forms:**

- `<form>` with `<input>` elements → React controlled components with `useState` or react-hook-form
- Form validation → keep simple with HTML5 attributes initially; error handling via try/catch added in the dimension pass (no Zod)

---

## Step 6: Wire routing

If the HTML had multiple pages:

- Each page becomes a route in react-router
- `<a href="about.html">` → `<Link to="/about">`
- Set up `src/routes/` with route definitions
- Add a layout component wrapping shared elements (nav, footer)

If single page: skip routing for now.

---

## Step 7: Verify visual fidelity

After conversion, the designer should see their design in the browser, built with React. Run `npm run dev` and visually compare:

- Colors should match (tokens from design.md → theme → components)
- Spacing should match
- Typography should match
- Component behavior (dropdowns open, modals appear, tabs switch) should work

If something looks wrong, the tokens or component mapping needs adjusting — not the design.

---

## Step 8: Hand off to the 18 dimensions

The project is now Vite + React + TypeScript with shadcn components and a design token system sourced from design.md. Run the normal vibe-to-prod dimension pass. Most dimensions should have a head start because:

- Dimension 2 (data extraction): mock data already moved to `src/data/`
- Dimension 4 (API stubs): fetch calls already moved to `api.ts`
- Dimension 8 (component library): shadcn used from the start
- Dimension 9 (design tokens): extracted from design.md into theme

The remaining dimensions (error boundaries, resilience, security, design quality) apply as normal.

---

## Component boundary decisions

The agent decides component boundaries using these rules:

1. Each HTML page = one page-level component
2. Visually distinct sections (header, sidebar, main, footer) = layout components
3. Repeated patterns (card grids, list items, table rows) = shared components
4. Interactive elements (modals, dropdowns, forms) = components with local state
5. If in doubt, keep it together — splitting can happen in the dimension 1 pass later

Don't over-componentize. A single page with a header, a form, and a table is three components, not fifteen.

---

## What NOT to change

- **The visual design.** Colors, spacing, layout — preserve exactly. If the HTML has a specific shade of blue, that shade goes into design.md and the theme. Don't "improve" it to a different blue.
- **Content hierarchy.** If the designer put the sidebar on the left, it stays on the left.
- **Intentional styling choices.** A large card with extra padding is a design decision, not a bug.

The conversion changes the technology. The design belongs to the designer.
