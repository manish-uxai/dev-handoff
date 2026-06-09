# Vibe-to-Prod

Universal [Agent Skill](https://agentskills.io) that transforms designer-built vibecoded prototypes into production-ready, frontend-only codebases — ready for backend teams to plug in and ship.

Works across **Claude Code**, **OpenAI Codex**, **Cursor**, **GitHub Copilot**, and any agent supporting the open `SKILL.md` format.

## The Problem

Designers build high-fidelity prototypes in code (Bolt, v0, Lovable, Replit, Claude Artifacts). The UI looks final, but the codebase is messy. Developers won't integrate it — they rebuild from Figma instead.

**Vibe-to-Prod breaks that cycle.** Harden the architecture under the designer's UI so a frontend engineer opens the project, plugs in APIs, and ships. Zero visual rewrites.

## What it does

The **18-Dimension Handoff Framework** (v3.1) enforces:

- Figma-less direct-to-code handoff — prototype IS the production UI
- Design intent preserved; implementation improved underneath
- shadcn/Radix replacement for reinvented primitives
- Typed `api.ts` stubs with `// @backend` markers + `BACKEND_CONTRACT.md`
- Evidence-based audits with required grep patterns
- JavaScript codebases flagged for TypeScript migration

See [`vibe-to-prod/SKILL.md`](vibe-to-prod/SKILL.md) for the full specification.

## Execution Modes

| Mode | Command | What it does |
|------|---------|--------------|
| **Audit** | `/vibe-to-prod audit` | Evidence-based report, no file changes |
| **Refactor** | `/vibe-to-prod refactor` | Full 18-dimension pass + BACKEND_CONTRACT.md |
| **Quick** | `/vibe-to-prod quick` | Structure + types + stubs only |

## Install

```bash
npx skills add manish-uxai/vibe-to-prod -y
```

### PromptScript warning

If you see `PromptScript does not support global skill installation` — skill still installed to `~/.agents/skills/vibe-to-prod`. Safe to ignore, or use:

```bash
npx skills add manish-uxai/vibe-to-prod -g -y -a universal
```

## Manual install

Copy `vibe-to-prod/` to your agent skills path. **Directory name must match `name: vibe-to-prod` in frontmatter.**

| Agent | Global | Project |
|-------|--------|---------|
| Cursor | `~/.cursor/skills/vibe-to-prod/` | `.cursor/skills/vibe-to-prod/` |
| Claude Code | `~/.claude/skills/vibe-to-prod/` | `.claude/skills/vibe-to-prod/` |
| Codex | `~/.codex/skills/vibe-to-prod/` | `.codex/skills/vibe-to-prod/` |
| GitHub Copilot | `~/.copilot/skills/vibe-to-prod/` | `.github/skills/vibe-to-prod/` |

## Repository structure

```
vibe-to-prod/                  # GitHub repo name (unchanged)
├── README.md
├── LICENSE
└── vibe-to-prod/             # Skill directory (matches frontmatter name)
    ├── SKILL.md
    └── references/
        ├── audit-checklist.md
        └── backend-contract-template.md
```

## License

MIT — see [LICENSE](LICENSE).
