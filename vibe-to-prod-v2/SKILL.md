---
name: vibe-to-prod-v2
description: Transforms designer-built React or Next.js prototypes into production-ready codebases. Modular architecture — detects stack and mode, then loads only what's needed. Use when a designer wants to make their vibe-coded UI production-ready, when asked to audit or refactor a prototype, when someone says "make this dev-ready", "clean up my prototype", "vibe to prod", or any mention of turning a coded prototype into something a developer can ship.
license: MIT
compatibility: Works with Claude Code, OpenAI Codex, Cursor, GitHub Copilot, and other agentskills.io-compatible agents.
metadata:
  author: vibe-to-prod
  version: "1.0.0"
  architecture: modular
---

# Router

This file does one thing: detect the context and tell you exactly which files to load. Read this file first, always. Then load only what's listed for your detected context.

---

## Step 1: Detect the mode

| User says | Mode | Load |
| :--- | :--- | :--- |
| `/vibe-to-prod-v2 audit` | Audit | `modes/audit.md` |
| `/vibe-to-prod-v2 refactor` | Refactor | `modes/refactor.md` |
| `/vibe-to-prod-v2 scaffold` | Scaffold | `modes/scaffold.md` |
| `/vibe-to-prod-v2 migrate` | Migration | `modes/migrate.md` |
| `/vibe-to-prod-v2 quick` | Quick | `modes/refactor.md` + note: dimensions 1, 2, 3, 4, 7, 17, 18 only |
| Anything else | Refactor | `modes/refactor.md` |

---

## Step 2: Detect the stack

Check `package.json`, file extensions, and folder structure.

| Detected | Load |
| :--- | :--- |
| `.tsx` / `.ts` files present | `stacks/react.md` — TypeScript path |
| `.jsx` / `.js` files, no `.ts` | `stacks/react.md` — JavaScript path |
| `next.config.*` or `next` in `package.json` | `stacks/nextjs.md` |
| Only `.html` / `.css`, no `package.json` | Stop. Read `core/constraints.md` for redirect response. |
| Vue, Svelte, Angular detected | Stop. Read `core/constraints.md` for redirect response. |

---

## Step 3: Always load these

Regardless of stack or mode, always load:

1. `core/language-rules.md`
2. `core/constraints.md`
3. `refs/smells.md`

---

## Step 4: Load dimensions based on scope

**Full audit or refactor** — load all dimension files:
- `dimensions/architecture.md`
- `dimensions/data-api.md`
- `dimensions/state-access.md`
- `dimensions/ui-quality.md`
- `dimensions/robustness.md`
- `dimensions/hygiene.md`

**Scoped request** — load only the matching file:

| User says | Load |
| :--- | :--- |
| "fix my components" / "architecture" | `dimensions/architecture.md` |
| "fix data flow" / "API stubs" / "mock data" | `dimensions/data-api.md` |
| "fix state" / "state management" / "context" | `dimensions/state-access.md` |
| "fix UI" / "primitives" / "design tokens" | `dimensions/ui-quality.md` |
| "fix resilience" / "error states" / "accessibility" | `dimensions/robustness.md` |
| "cleanup" / "hygiene" / "icons" / "unused files" | `dimensions/hygiene.md` |

For scoped requests, state which dimensions you are applying at the start of your response.

---

## What this file does NOT contain

- No dimension rules (those are in `dimensions/`)
- No tone or language rules (those are in `core/language-rules.md`)
- No grep patterns (those are in `modes/audit.md`)
- No stack-specific rules (those are in `stacks/`)

If you find yourself applying a rule from memory rather than a loaded file, stop and load the relevant file first.
