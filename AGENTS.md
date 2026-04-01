# AGENTS.md

This repository captures reusable skills for AI coding agents, following the [agent skills standard](https://claude.ai/code). Skills are self-contained Markdown files that agents can invoke to perform common, well-defined tasks.

## Repository Purpose

- Store reusable, composable skills for use with Claude Code and other AI coding agents
- Provide clear, consistent conventions so agents can discover and apply skills reliably
- Serve as a reference implementation of the agent skills standard

## Structure

```
skills/
├── AGENTS.md          # This file — agent guidance for the repo
├── CLAUDE.md          # Symlink → AGENTS.md (Claude Code compatibility)
├── LICENSE
└── <skill-name>.md    # Each skill is a single Markdown file
```

## Skills Format

Each skill file (`<skill-name>.md`) follows this structure:

```markdown
---
description: One-line description of what the skill does and when to invoke it
---

# <Skill Name>

Brief overview of the skill's purpose.

## Usage

How the skill is invoked and any arguments it accepts.

## Steps

Numbered or bulleted steps the agent should follow.

## Examples

Concrete examples demonstrating the skill in action.
```

Key conventions:
- Filenames are lowercase, hyphen-separated (e.g., `review-pr.md`, `commit.md`)
- Each skill is self-contained — no cross-file dependencies unless explicitly stated
- Steps should be actionable and unambiguous; prefer commands over prose
- Include real examples over abstract descriptions

## Adding a New Skill

1. Create `<skill-name>.md` at the repo root
2. Add a `description:` frontmatter field — this is what agents use to match triggers
3. Write clear, concrete steps (executable commands > paragraphs)
4. Commit with a descriptive message: `add <skill-name> skill: <one-line summary>`

## Git Workflow

- Default branch: `main`
- Feature branches: `<author>/<short-description>` or `claude/<task-slug>`
- Commit messages: imperative mood, lowercase, no trailing period
- Do not force-push to `main`

## Code Style & Conventions

- Markdown: use ATX headings (`#`), fenced code blocks with language tags
- Frontmatter: YAML, delimited by `---`
- Keep skill files focused — one skill per file, no bundling unrelated tasks
- Prefer explicit over implicit; agents should not need to infer intent

## Security

- Never commit secrets, tokens, or credentials
- Skills that touch external services must document required environment variables
- Do not add skills that perform destructive operations without explicit confirmation steps
