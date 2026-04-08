# AGENTS.md

This repository captures reusable skills for AI coding agents. Agent guidance follows the [AGENTS.md standard](https://agents.md); skill format follows the [Agent Skills standard](https://agentskills.io).

## Repository Purpose

- Store reusable, composable skills for use with Claude Code and other AI coding agents
- Provide clear, consistent conventions so agents can discover and apply skills reliably
- Follow the [agentskills.io](https://agentskills.io) open standard for skill format and discovery

## Structure

```
├── AGENTS.md              # This file — agent guidance (agents.md standard)
├── CHANGELOG.md           # Version history
├── LICENSE
├── README.md              # Installation and usage reference
├── .claude/
│   └── skills -> ../.agents/skills   # Symlink for standalone Claude Code use
├── .claude-plugin/
│   └── plugin.json        # Claude Code plugin manifest
└── .agents/
    └── skills/
        └── <skill-name>/          # Each skill is a directory
            ├── SKILL.md           # Required: YAML frontmatter + instructions
            ├── scripts/           # Optional: executable scripts (Python, Bash, JS)
            ├── references/        # Optional: reference materials, API docs, schemas
            └── assets/            # Optional: templates, examples, other resources
```

Skills live under `.agents/skills/` so that agents running within this repository discover them automatically via standard agent runtime conventions.

## Plugin Installation

This repository ships a Claude Code plugin manifest at `.claude-plugin/plugin.json`. The manifest points to `.agents/skills/` and registers skills under the `skills` namespace (e.g. `/skills:manage-plans`).

**Install via plugin directory:**

```sh
claude --plugin-dir /path/to/skills
```

**Install from the repository URL** (once Claude Code supports remote plugin installation):

```sh
claude plugin install https://github.com/geemus/skills
```

**Standalone use** (without plugin namespace): the `.claude/skills` symlink provides backward-compatible skill discovery for agents running within this repository.

## Skills Format

Each skill is a **directory** named after the skill containing a `SKILL.md` file with YAML frontmatter and Markdown instructions.

For full format rules, naming constraints, progressive disclosure levels, and a worked example, read `.agents/skills/manage-skills/references/skill-format.md`. To create, update, or delete skills using guided steps, use the **`manage-skills`** skill (`/manage-skills`).

## Available Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `manage-skills` | `/manage-skills` | Create, update, manage, and audit skills in this repository |
| `manage-plans` | `/manage-plans` | Create and update structured work plans as GitHub issues |
| `format-review-comments` | applied proactively during code review | Format review and feedback comments using the Conventional Comments standard |
| `review-pr` | `/review-pr` | Conduct a systematic PR review across logic, security, performance, test coverage, and documentation |
| `review-skill` | `/review-skill` | Evaluate a skill for effectiveness across instruction clarity, trigger discoverability, example completeness, composability, and scope coherence |
| `create-commit` | `/commit` | Stage changes safely, generate a conventional-commit message, and block secrets from being committed |
| `refine-prose` | `/refine-prose` | Polish drafted prose for clarity, conciseness, and consistent voice before presenting or posting |
| `manage-sprites` | `/manage-sprites` | Provision, operate, checkpoint, and destroy sprites.dev instances via the `sprite` CLI |

## Adding or Managing Skills

To create, update, or delete skills with guided steps and correct conventions, use the **`manage-skills`** skill (`/manage-skills`). To learn the skill format first, read `.agents/skills/manage-skills/references/skill-format.md`.

## Git Workflow

- Default branch: `main`
- Feature branches: `<author>/<short-description>` or `claude/<task-slug>`
- **PR workflow**: When a PR is the goal, always create a feature branch first — never commit directly to `main` and open a PR from there. Branch, commit, push the branch, then `gh pr create`.
- Commit messages: [Conventional Commits](https://www.conventionalcommits.org) format — `<type>[optional scope]: <description>` — where description is lowercase imperative mood with no trailing period
  - Valid types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`
  - Scope (optional): the skill name or component affected, e.g. `feat(plan):`, `fix(skills):`
  - Examples:
    - `docs: adopt conventional commits standard`
    - `feat(manage-skills): add loop skill`
    - `fix(manage-plans): correct frontmatter validation logic`
    - `chore: update AGENTS.md structure`
- Do not force-push to `main`

## Code Style & Conventions

- Markdown: ATX headings (`#`), fenced code blocks with language tags
- Frontmatter: YAML, delimited by `---`
- One skill per directory; do not bundle unrelated tasks
- Prefer explicit over implicit; agents should not need to infer intent
- Scripts should be self-contained or clearly document their dependencies

## Security

- Never commit secrets, tokens, or credentials
- Skills that touch external services must document required environment variables in their `SKILL.md`
- Do not add skills that perform destructive operations without explicit confirmation steps
- Only use skills from trusted sources — treat them like installed software
