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

Skills are available under the `skills` namespace (e.g., `/skills:upsert-plan`) when installed via the Claude Code plugin, or directly (e.g., `/upsert-plan`) when running within this repository.

## Skills Format

Each skill is a **directory** named after the skill containing a `SKILL.md` file with YAML frontmatter and Markdown instructions.

For full format rules, naming constraints, progressive disclosure levels, and a worked example, read `.agents/skills/upsert-skill/references/skill-format.md`. To create, update, or delete skills using guided steps, use the **`upsert-skill`** skill (`/upsert-skill`).

## Available Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `upsert-skill` | `/upsert-skill` | Create, update, manage, and audit skills in this repository |
| `upsert-plan` | `/upsert-plan` | Create and update structured work plans as GitHub issues |
| `format-review-comments` | apply automatically when writing any review comment, PR feedback, or critique of code | Format review and feedback comments using the Conventional Comments standard |
| `review-pr` | `/review-pr` | Conduct a systematic PR review across logic, security, performance, test coverage, and documentation |
| `review-agents-md` | `/review-agents-md` | Audit AGENTS.md and equivalent agent instruction files for completeness, accuracy, agent-friendliness, freshness, and scope coherence |
| `review-skill` | `/review-skill` | Evaluate a skill for effectiveness across instruction clarity, trigger discoverability, example completeness, composability, and scope coherence |
| `create-commit` | `/commit` | Stage changes safely, generate a conventional-commit message, and block secrets from being committed |
| `refine-prose` | `/refine-prose` | Polish drafted prose for clarity, conciseness, and consistent voice before presenting or posting |
| `manage-sprite` | `/manage-sprite` | Provision, operate, checkpoint, and destroy sprites.dev instances via the `sprite` CLI |
| `manage-sprite-env` | `/manage-sprite-env` | Manage services, checkpoints, and environment info from within a running sprite instance |
| `audit-skills` | `/audit-skills` | Produce a full health report on the skill library: structural validation per skill and cross-skill analysis covering trigger conflicts, redundancy, workflow gaps, and composition chains |

## Adding or Managing Skills

To create, update, or delete skills with guided steps and correct conventions, use the **`upsert-skill`** skill (`/upsert-skill`). To learn the skill format first, read `.agents/skills/upsert-skill/references/skill-format.md`.

## Git Workflow

- Default branch: `main`
- Feature branches: `<author>/<short-description>` or `claude/<task-slug>`
- **PR workflow**: When a PR is the goal, always create a feature branch first — never commit directly to `main` and open a PR from there. Branch, commit, push the branch, then `gh pr create`.
- Commit messages: [Conventional Commits](https://www.conventionalcommits.org) format — `<type>[optional scope]: <description>` — where description is lowercase imperative mood with no trailing period
  - Valid types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`
  - Scope (optional): the skill name or component affected, e.g. `feat(plan):`, `fix(skills):`
  - Examples:
    - `docs: adopt conventional commits standard`
    - `feat(upsert-skill): add loop skill`
    - `fix(upsert-plan): correct frontmatter validation logic`
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

## Environment

The following CLI tools are required for full functionality:

- `gh` — GitHub CLI, used for PR creation (`gh pr create`) in the Git Workflow
- `sprite` — sprites.dev CLI, used by the `manage-sprite` skill
