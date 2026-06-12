# vibe-to-prod

**Turn your vibe-coded prototype into production-ready code your developer can actually ship.**

You designed something great — in Figma Make, with an AI agent, or by hand. Now it needs to become real software. vibe-to-prod hardens your prototype into clean, typed, production-ready React so a developer can plug in real data and ship it, instead of rebuilding your UI from scratch.

No terminal expertise required. You talk to your AI agent in plain English; the skill does the rest.

---

## What it does

vibe-to-prod takes the UI you've already built and makes it production-grade:

- **Connects your data properly** — moves scattered data logic into one clean layer a developer can swap for real APIs in one place
- **Always outputs TypeScript** — gives your developer type safety and autocomplete; if your code is JavaScript or plain HTML, it's converted automatically
- **Uses real components** — replaces hand-rolled UI with shadcn/Radix components developers trust
- **Keeps your design** — your colors, spacing, and layout are preserved exactly; only the code underneath changes
- **Captures your design system** — extracts a `design.md` that becomes the single source of truth for your brand
- **Checks 19 production dimensions** — security, accessibility, error & empty states, routing, design quality, and more
- **Hands off cleanly** — your developer opens the project and starts integrating immediately

The result: weeks of developer rebuild time saved.

---

## Who it's for

**Designers and PMs** who build UIs with AI tools (Figma Make, Cursor, Copilot, v0) and need to hand them to developers. This is the primary audience — you don't need to know how to code. You describe what you want; the skill makes it production-ready.

**Developers** who inherit designer-built prototypes and want them hardened before integration, or who want a fast path from a vibe-coded proof-of-concept to a clean codebase.

---

## Installation

vibe-to-prod is an [agentskills.io](https://agentskills.io)-compatible skill. It works with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, and other compatible agents.

```bash
npx skills add Manish-UXUI/vibe-to-prod
```

You'll need [Node.js](https://nodejs.org) installed. If you don't have it, the skill will detect that and walk you through installing it.

---

## Getting started

Open your project in your IDE (VS Code, Cursor, etc.), then talk to your AI agent.

### Starting fresh

```
/vibe-to-prod start
```

Sets up a complete production-ready project (Vite + React + TypeScript + shadcn) and installs helpful companion skills. Start building features and they'll be production-ready from day one.

### You have existing code (Figma Make, HTML, or React)

```
/vibe-to-prod
```

The skill detects what you have and hardens it. If it's JavaScript, it migrates to TypeScript first. If it's plain HTML, it converts to React. Then it runs the full production pass.

Just say **"fix it all"** and it works through everything.

### You want to see what needs fixing first

```
/vibe-to-prod audit
```

Produces a report — what's production-ready, what isn't, and a plain-language summary written for designers. Nothing is changed; it's a read-only health check.

---

## The modes

| Command                                     | What it does                                                                        |
| ------------------------------------------- | ----------------------------------------------------------------------------------- |
| `/vibe-to-prod start`                       | Greenfield — scaffold a new project, install companion skills, set up design system |
| `/vibe-to-prod` or `/vibe-to-prod refactor` | Harden existing code (the default)                                                  |
| `/vibe-to-prod audit`                       | Read-only report of production readiness                                            |
| `/vibe-to-prod scaffold`                    | Set up project structure only                                                       |
| `/vibe-to-prod quick`                       | Fast pass — structure, types, and data layer only                                   |

---

## design.md — your design system, in your language

vibe-to-prod creates a `design.md` in your project: a human-readable document describing your colors, typography, spacing, and components. It's written in design language, not code.

This file is the **source of truth**. The code's styling is generated from it. When you update `design.md`, the styles update to match. You can even paste a `design.md` from another project to apply a whole new design system.

---

## What gets checked (the 19 dimensions)

The skill evaluates your code across 19 production dimensions, grouped into:

- **Architecture & data** — component structure, clean data extraction, typed API stubs, state management
- **UI quality** — design tokens, component library compliance, no AI-slop styling, performance
- **Robustness** — accessibility, error boundaries, loading/empty states, production resilience
- **Handoff & security** — dependency hygiene, onboarding setup, security basics, design quality

Every finding explains what it is and why it matters — so you learn as you go, growing from designer toward design engineer.

---

## Companion files it creates

| File            | Purpose                                                                        |
| --------------- | ------------------------------------------------------------------------------ |
| `design.md`     | Your design system source of truth                                             |
| `guidelines.md` | Conventions your AI agent follows on every future feature, so code stays clean |

After setup, add this to your agent's instructions so it stays on track:

> "Read guidelines.md and design.md before building any feature. Follow the code conventions and use the design tokens."

---

## Requirements

- **Node.js** — required to run React projects. [Download here](https://nodejs.org) if you don't have it.
- **An agentskills.io-compatible agent** — Claude Code, Codex, Cursor, Copilot, etc.

---

## Output is always TypeScript

vibe-to-prod always produces TypeScript. JavaScript and HTML inputs are converted automatically — no choice to make, no question asked. TypeScript is the production-handoff standard: it gives developers type safety, autocomplete, and confidence when refactoring. (If you specifically need a JavaScript-only output, this skill isn't the right fit.)

---

## License

MIT

---

## Feedback

vibe-to-prod is built by a designer, for designers, with developers in mind. If something doesn't work the way you expect — or you have an idea — open an issue on the repo.
