# Dev-Handoff

Universal [Agent Skill](https://agentskills.io) that transforms messy vibecoded prototypes into production-ready, enterprise-grade frontend codebases — ready for backend teams to plug in and ship.

Works across **Claude Code**, **OpenAI Codex**, **Cursor**, **GitHub Copilot**, and any agent supporting the open `SKILL.md` format.

## The Problem

Designers and product teams use vibecoding tools (Bolt, v0, Lovable, Replit, Claude Artifacts) to prototype working UIs fast. The output *looks* right but the code is unmaintainable: god-components, inline mock data, no types, no structure. Developers can't use it.

**Dev-Handoff bridges that gap.** It refactors vibecoded prototypes into clean, typed, well-structured frontend code with clearly marked backend integration points — so devs receive it and just swap stubs for real APIs.

## What it does

The **16-Dimension Handoff Framework** enforces:

- Component architecture, data extraction, and canonical domain types
- Typed `api.ts` stubs with `// @backend` markers and generated `BACKEND_CONTRACT.md`
- State management, RBAC, routing, memoization, a11y
- CSS token compliance, destructive-action guards, error boundaries
- Dependency hygiene, file hygiene, and visual fidelity preservation

See [`dev-handoff/SKILL.md`](dev-handoff/SKILL.md) for the full specification.

## Execution Modes

| Mode | Command | What it does |
|------|---------|--------------|
| **Audit** | `/dev-handoff audit` | Report violations without changing code |
| **Refactor** | `/dev-handoff refactor` | Full 16-dimension pass + BACKEND_CONTRACT.md |
| **Quick** | `/dev-handoff quick` | Structure + types + stubs only (fastest handoff) |

## Install

```bash
npx skills add Manish-UXUI/dev-handoff -g -y
```

Project-only (PromptScript, or team-shared in repo):

```bash
npx skills add Manish-UXUI/dev-handoff -y
```

### PromptScript / "Failed to install 1" warning

If you see:

```text
✗ dev-handoff → PromptScript: PromptScript does not support global skill installation
```

**The skill still installed** to `~/.agents/skills/dev-handoff` — Cursor, Copilot, Claude Code, and Codex all read that path. Safe to ignore.

To install globally **without** the warning:

```bash
npx skills add Manish-UXUI/dev-handoff -g -y -a universal
```

## Manual install by platform

Copy the `dev-handoff/` folder to the path your agent expects:

| Agent | Personal (global) | Project (repo) |
|-------|-------------------|----------------|
| **Cursor** | `~/.cursor/skills/dev-handoff/` | `.cursor/skills/dev-handoff/` |
| **Claude Code** | `~/.claude/skills/dev-handoff/` | `.claude/skills/dev-handoff/` |
| **OpenAI Codex** | `~/.codex/skills/dev-handoff/` | `.codex/skills/dev-handoff/` |
| **GitHub Copilot** | `~/.copilot/skills/dev-handoff/` | `.github/skills/dev-handoff/` |
| **Universal fallback** | `~/.agents/skills/dev-handoff/` | `.agents/skills/dev-handoff/` |

**Important:** The directory name must match the `name` field in frontmatter (`dev-handoff`).

## Usage

The skill activates automatically when you ask for handoff-related work, or invoke explicitly:

| Agent | Example |
|-------|---------|
| Claude Code | `/dev-handoff audit src/pages/Dashboard.tsx` |
| Cursor | "Use dev-handoff to refactor this prototype for backend handoff" |
| Codex | `$dev-handoff quick src/App.tsx` |
| Copilot (VS Code) | "Audit this vibecoded prototype against the 16-dimension framework" |

### Example prompts

```
Audit this React prototype against the 16-dimension handoff framework.

Refactor this vibecoded app for backend handoff: extract mocks, add domain.ts
and api.ts with @backend annotations, generate BACKEND_CONTRACT.md.

Quick clean this prototype — structure, types, and stubs only.
I need devs to start integrating by tomorrow.
```

## Repository structure

```
Dev-Handoff/
├── README.md
├── LICENSE
└── dev-handoff/
    ├── SKILL.md                               # Agent instructions (16 dimensions)
    └── references/
        ├── audit-checklist.md                 # Full-repo audit checklist + grep patterns
        └── backend-contract-template.md       # Template for generated handoff doc
```

## Compatibility

Built to the [Agent Skills open standard](https://agentskills.io):

- YAML frontmatter (`name`, `description`) for agent discovery
- Markdown body for instructions
- `references/` for progressive disclosure

Validated against: Claude Code, Cursor, Codex CLI, GitHub Copilot (VS Code / Visual Studio).

## License

MIT — see [LICENSE](LICENSE).
