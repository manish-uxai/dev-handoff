# design.md Template

> This is the structure the agent uses when creating a `design.md` for a project. design.md is the **design system source of truth** — written in design language, structured so an agent can build new on-brand pages without seeing existing code. The theme CSS tokens are generated FROM this file.
>
> Fill each section from what actually exists in the project. Don't invent tokens that aren't represented. If a section can't be filled (e.g. no responsive info in a static export), write what's there and note it under Known Gaps.
>
> Use `{token.name}` syntax throughout — it's the bridge to the theme CSS. The agent reads `{colors.primary}` here and generates `--primary: #hex` in the theme.

---

## Overview

A few sentences capturing the brand feel and positioning: what's the atmosphere (warm, clean, technical, playful), how does it want to feel, what makes it distinct.

**Key Characteristics:**

- The signature color and how it's used
- The type treatment (serif/sans, display vs body split)
- The surface logic (light, dark, alternating)
- The spacing/rhythm feel
- Any defining visual motif

---

## Colors

Group by role. Each entry: token, hex, and a usage note explaining WHERE and WHY.

### Brand & Accent

- **Primary** (`{colors.primary}` — #**\_\_**): The signature brand color. Used on [primary CTAs, links, active states].
- **Primary Active** (`{colors.primary-active}` — #**\_\_**): Press/hover-darker variant.
- **Accent** (`{colors.accent}` — #**\_\_**): Secondary accent, used sparingly on [...].

### Surface

- **Canvas** (`{colors.canvas}` — #**\_\_**): Default page background.
- **Surface Soft** (`{colors.surface-soft}` — #**\_\_**): Section dividers, soft bands.
- **Surface Card** (`{colors.surface-card}` — #**\_\_**): Card backgrounds, one step off canvas.
- **Surface Dark** (`{colors.surface-dark}` — #**\_\_**): Dark product surfaces, if any.

### Text

- **Ink** (`{colors.ink}` — #**\_\_**): Headlines and primary text.
- **Body** (`{colors.body}` — #**\_\_**): Default running text.
- **Muted** (`{colors.muted}` — #**\_\_**): Secondary text, captions.
- **On Primary** (`{colors.on-primary}` — #**\_\_**): Text on primary-color buttons.

### Semantic

- **Success** (`{colors.success}` — #**\_\_**): Status indicators, confirmations.
- **Warning** (`{colors.warning}` — #**\_\_**): Warning callouts.
- **Error** (`{colors.error}` — #**\_\_**): Validation errors, destructive actions.

---

## Typography

### Font Family

Display: [font] for headlines. Body: [font] for text/UI. Mono: [font] for code. Note any fallback stack and substitutes if the primary font is licensed/unavailable.

### Hierarchy

| Token                     | Size   | Weight | Line Height | Letter Spacing | Use              |
| ------------------------- | ------ | ------ | ----------- | -------------- | ---------------- |
| `{typography.display-xl}` | \_\_px | \_\_\_ | \_\_\_      | \_\_\_         | Page h1          |
| `{typography.display-lg}` | \_\_px | \_\_\_ | \_\_\_      | \_\_\_         | Section heads    |
| `{typography.title-md}`   | \_\_px | \_\_\_ | \_\_\_      | \_\_\_         | Card titles      |
| `{typography.body-md}`    | \_\_px | \_\_\_ | \_\_\_      | \_\_\_         | Running text     |
| `{typography.caption}`    | \_\_px | \_\_\_ | \_\_\_      | \_\_\_         | Captions, labels |
| `{typography.code}`       | \_\_px | \_\_\_ | \_\_\_      | \_\_\_         | Code blocks      |
| `{typography.button}`     | \_\_px | \_\_\_ | \_\_\_      | \_\_\_         | Button labels    |

### Principles

Non-obvious rules: weight conventions, serif-vs-sans usage, tracking rules, anything that keeps type on-brand.

---

## Layout

### Spacing System

- Base unit: \_\_px
- Tokens: `{spacing.xs}` **px · `{spacing.sm}` **px · `{spacing.md}` **px · `{spacing.lg}` **px · `{spacing.xl}` **px · `{spacing.section}` **px
- Section padding, card padding, callout padding values.

### Grid & Container

- Max content width
- Grid column logic
- Feature/card grid behavior at desktop/tablet/mobile

### Whitespace Philosophy

How the spacing creates the brand's pacing — tight and dense, or generous and editorial.

---

## Elevation & Depth

| Level    | Treatment                     | Use                   |
| -------- | ----------------------------- | --------------------- |
| Flat     | No shadow, no border          | Body sections         |
| Hairline | 1px border                    | Inputs, sub-nav       |
| Card     | Surface background, no shadow | Feature cards         |
| Shadow   | Faint drop shadow             | Hover-elevated states |

Note whether the system is shadow-heavy or color-block-first.

---

## Shapes

### Border Radius Scale

| Token            | Value  | Use                           |
| ---------------- | ------ | ----------------------------- |
| `{rounded.sm}`   | \_\_px | Small buttons, dropdown items |
| `{rounded.md}`   | \_\_px | Buttons, inputs               |
| `{rounded.lg}`   | \_\_px | Content cards                 |
| `{rounded.xl}`   | \_\_px | Hero containers               |
| `{rounded.pill}` | 9999px | Badges, pills                 |

### Illustration / Photography

Style notes if any — line-art, product mockups, photography treatment, avatar cropping.

---

## Components

Each major component with token references, dimensions, and variants. This is the most valuable section — it lets an agent build on-brand without seeing the code.

**`button-primary`** — Background `{colors.primary}`, text `{colors.on-primary}`, `{typography.button}`, padding **×**, height \_\_px, rounded `{rounded.md}`. Active state darkens to `{colors.primary-active}`.

**`button-secondary`** — Background `{colors.canvas}`, text `{colors.ink}`, 1px hairline border, same dimensions as primary.

**`feature-card`** — Background `{colors.surface-card}`, rounded `{rounded.lg}`, padding `{spacing.xl}`. Icon, `{typography.title-md}` headline, `{typography.body-md}` body.

**`text-input`** — Background `{colors.canvas}`, text `{colors.ink}`, `{typography.body-md}`, rounded `{rounded.md}`, 1px hairline border. Focus state adds a primary-color ring.

[Add every recurring component: cards, dialogs, badges, tabs, nav, footer, etc.]

---

## Do's and Don'ts

### Do

- The non-negotiable brand rules — anchor color, type treatment, surface rhythm.

### Don't

- The anti-patterns — what breaks the brand, what to never introduce.

---

## Responsive Behavior

| Breakpoint | Width      | Key Changes                                 |
| ---------- | ---------- | ------------------------------------------- |
| Mobile     | < 768px    | [nav collapse, grid stacking, type scaling] |
| Tablet     | 768–1024px | [column reductions]                         |
| Desktop    | > 1024px   | [full layout]                               |

Collapsing strategy: how grids and components adapt down.

---

## Known Gaps

- Fonts that are licensed/unavailable and their substitutes
- Logo/glyph assets that are images, not tokens
- Anything out of scope (animation timings, product-specific components)
- Sections that couldn't be fully extracted from the source
