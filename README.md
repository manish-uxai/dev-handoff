# Dev-Handoff

Universal [Agent Skill](https://agentskills.io) for auditing and refactoring MVP prototypes ("vibecodes") into production-ready, enterprise-grade frontend codebases.

Works across **Claude Code**, **OpenAI Codex**, **Cursor**, **GitHub Copilot**, and any other agent that supports the open `SKILL.md` format.

## What it does

The **15-Dimension Handoff Framework** enforces:

- Component architecture and domain isolation
- Extracted mock/seed data and canonical `domain.ts` types
- Typed `api.ts` contract stubs
- Enterprise state management, RBAC, and destructive-action guards
- CSS token compliance, routing, memoization, a11y, strict TypeScript, and error boundaries

See [`dev-handoff/SKILL.md`](dev-handoff/SKILL.md) for the full specification.

## Install (recommended)

Use the [Skills CLI](https://skills.sh/) — one command, all supported agents:

```bash
npx skills add Manish-UXUI/dev-handoff@dev-handoff -g -y
```

Install into the current project only (omit `-g`):

```bash
npx skills add Manish-UXUI/dev-handoff@dev-handoff -y
```

## Manual install by platform

All platforms read the same `dev-handoff/SKILL.md`. Copy the `dev-handoff/` folder to the path your agent expects.

| Agent | Personal (global) | Project (repo) |
|-------|-------------------|----------------|
| **Cursor** | `~/.cursor/skills/dev-handoff/` | `.cursor/skills/dev-handoff/` |
| **Claude Code** | `~/.claude/skills/dev-handoff/` | `.claude/skills/dev-handoff/` |
| **OpenAI Codex** | `~/.codex/skills/dev-handoff/` | `.codex/skills/dev-handoff/` |
| **GitHub Copilot** | `~/.copilot/skills/dev-handoff/` | `.github/skills/dev-handoff/` |
| **Universal fallback** | `~/.agents/skills/dev-handoff/` | `.agents/skills/dev-handoff/` |

**Important:** The directory name must match the `name` field in frontmatter (`dev-handoff`).

## Usage

The skill activates automatically when you ask for handoff-related work, or invoke it explicitly:

| Agent | Example invocation |
|-------|-------------------|
| Claude Code | `/dev-handoff audit src/pages/Dashboard.tsx` |
| Cursor | "Use the dev-handoff skill to audit this prototype" |
| Codex | `$dev-handoff refactor src/App.tsx for backend handoff` |
| Copilot (VS Code) | "Audit this file against the dev-handoff 15-dimension framework" |

### Example prompts

```
Audit this React prototype against the 15-dimension handoff framework.
List violations before refactoring.

Refactor src/App.tsx for backend handoff: extract mocks to /src/data/,
add domain.ts and api.ts stubs, and replace conditional routing with react-router.

Prepare this vibecode for enterprise handoff. Enforce RBAC, ConfirmDialog
on deletes, CSS tokens, and error boundaries.
```

## Repository structure

```
Dev-Handoff/
├── README.md
├── LICENSE
└── dev-handoff/
    ├── SKILL.md                      # Required — agent instructions
    └── references/
        └── audit-checklist.md        # Optional — full-repo audit checklist
```

## Compatibility

Built to the [Agent Skills open standard](https://agentskills.io):

- YAML frontmatter (`name`, `description`) for agent discovery
- Markdown body for instructions
- Optional `references/` for progressive disclosure

Validated against: Claude Code, Cursor, Codex CLI, GitHub Copilot (VS Code / Visual Studio).

## License

MIT — see [LICENSE](LICENSE).
