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
├── CLAUDE.md              # Symlink → AGENTS.md (Claude Code compatibility)
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

Each skill is a **directory** named after the skill. The directory must contain a `SKILL.md` file with YAML frontmatter followed by Markdown instructions. See the [Agent Skills specification](https://agentskills.io/specification) for full details.

### Frontmatter

```yaml
---
name: skill-name          # required: must match the directory name
description: >            # required: what this skill does and when to use it
  Extracts and summarizes content from PDF files. Use when the user
  asks to read, parse, or extract text from a PDF document.
license: Apache-2.0       # optional
metadata:                 # optional: version tracking, authorship
  author: your-handle
  version: "1.0"
allowed-tools:            # optional: restrict which tools the skill may use
  - Bash
  - Read
---
```

**`name`** rules:
- Lowercase letters, numbers, and hyphens only
- 1–64 characters; no leading/trailing/consecutive hyphens
- Must match the parent directory name exactly

**`description`** rules:
- Non-empty, maximum 1024 characters
- Write in third person — this text is injected into the system prompt for discovery
- Include both *what* the skill does and *when* to invoke it

### Markdown body

Write whatever helps agents perform the task effectively. Recommended sections:

```markdown
# Skill Name

Brief overview of the skill's purpose.

## Instructions

Step-by-step guidance for the agent to follow.

## Examples

Concrete examples with inputs and expected outputs.
```

### Progressive disclosure

Skills load in three stages — keep each lean:

| Level | Content | When loaded | Target size |
|-------|---------|-------------|-------------|
| 1 | `name` + `description` frontmatter | Always (startup) | ~100 tokens |
| 2 | `SKILL.md` body | When skill is triggered | < 5,000 tokens |
| 3 | `scripts/`, `references/`, `assets/` | On demand, via bash | Unlimited |

## Adding a New Skill

1. Create a directory: `.agents/skills/<skill-name>/` (lowercase, hyphen-separated)
2. Add `.agents/skills/<skill-name>/SKILL.md` with `name:` and `description:` frontmatter
3. Write clear, concrete instructions in the Markdown body
4. Add `scripts/`, `references/`, or `assets/` as needed for Level 3 content
5. Commit: `add <skill-name> skill: <one-line summary>`

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
