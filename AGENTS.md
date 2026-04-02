# AGENTS.md

This repository captures reusable skills for AI coding agents. Agent guidance follows the [AGENTS.md standard](https://agents.md); skill format follows the [Agent Skills standard](https://agentskills.io).

## Repository Purpose

- Store reusable, composable skills for use with Claude Code and other AI coding agents
- Provide clear, consistent conventions so agents can discover and apply skills reliably
- Follow the [agentskills.io](https://agentskills.io) open standard for skill format and discovery

## Structure

```
skills/
├── AGENTS.md              # This file — agent guidance (agents.md standard)
├── LICENSE
└── .agents/
    └── skills/
        └── <skill-name>/          # Each skill is a directory
            ├── SKILL.md           # Required: YAML frontmatter + instructions
            ├── scripts/           # Optional: executable scripts (Python, Bash, JS)
            ├── references/        # Optional: reference materials, API docs, schemas
            └── assets/            # Optional: templates, examples, other resources
```

Skills live under `.agents/skills/` so that agents running within this repository discover them automatically via standard agent runtime conventions.

## Skills Format

Each skill is a **directory** named after the skill containing a `SKILL.md` file with YAML frontmatter and Markdown instructions.

For full format rules, naming constraints, progressive disclosure levels, and a worked example, use the **`skills`** skill or read `.agents/skills/skills/references/skill-format.md` directly.

## Available Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `skills` | `/skills` | Create, update, and manage skills in this repository |
| `plan` | `/plan` | Generate a structured work plan and write it to a GitHub issue |

## Adding or Managing Skills

Use the **`skills`** skill — it covers creating, updating, and deleting skills with the correct conventions and commit messages.

## Git Workflow

- Default branch: `main`
- Feature branches: `<author>/<short-description>` or `claude/<task-slug>`
- Commit messages: imperative mood, lowercase, no trailing period
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
