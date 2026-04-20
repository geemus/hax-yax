# AGENTS.md

This repository stores reusable skills for Claude Code and other AI coding agents. Agent guidance follows the [AGENTS.md standard](https://agents.md); skill format follows the [Agent Skills standard](https://agentskills.io).

## Repository Purpose

This repository stores reusable, composable skills for Claude Code and other AI coding agents. Skills follow the [agentskills.io](https://agentskills.io) open standard and ship via a native Claude Code plugin. The primary use cases are discovering available skills, creating new ones, updating existing ones, and composing skills into workflows.

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

Skills are available under the `hax-yax` namespace (e.g., `/hax-yax:upsert-plan`) when installed via the Claude Code plugin, or directly (e.g., `/upsert-plan`) when running within this repository.

## Skills Format

Each skill is a **directory** named after the skill containing a `SKILL.md` file with YAML frontmatter and Markdown instructions.

For full format rules, naming constraints, progressive disclosure levels, and a worked example, read `.agents/skills/upsert-skill/references/skill-format.md`. To create, update, or delete skills using guided steps, use the **`upsert-skill`** skill (`/upsert-skill`).

## Available Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `audit-skills` | `/audit-skills` | Produce a full health report on the skill library: structural validation per skill and cross-skill analysis covering trigger conflicts, redundancy, workflow gaps, and composition chains |
| `create-commit` | `/commit` | Stage changes safely, generate a conventional-commit message, and block secrets from being committed |
| `format-review-comments` | apply automatically when writing any review comment, PR feedback, or critique of code | Format review and feedback comments using the Conventional Comments standard |
| `implement-plan` | `/implement-plan` | Fetch a structured plan from a GitHub issue and orchestrate the full delivery workflow: step through unchecked tasks in dependency order, commit via create-commit, review via review-pr, and open a PR linked to the plan issue |
| `refine-prose` | `/refine-prose` | Polish drafted prose for clarity, conciseness, and consistent voice before presenting or posting |
| `review-agents-md` | `/review-agents-md` | Audit AGENTS.md and equivalent agent instruction files for completeness, accuracy, agent-friendliness, freshness, and scope coherence |
| `review-plan` | `/review-plan` | Evaluate a structured work plan across six quality dimensions: objective clarity, task actionability, dependency correctness, acceptance criteria completeness, scope appropriateness, and structural completeness |
| `review-pr` | `/review-pr` | Conduct a systematic PR review across logic, security, performance, test coverage, and documentation |
| `review-skill` | `/review-skill` | Evaluate a skill for effectiveness across instruction clarity, trigger discoverability, example completeness, composability, and scope coherence |
| `upsert-agents-md` | `/upsert-agents-md` | Create or update AGENTS.md and equivalent agent instruction files by surveying the repository |
| `upsert-plan` | `/upsert-plan` | Create and update structured work plans as GitHub issues |
| `upsert-skill` | `/upsert-skill` | Manage the lifecycle of reusable agent skills stored under `.agents/skills/` |

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

## Worktree Awareness

When running inside a git worktree (secondary working tree), scope all file operations to the current worktree root.

- **Resolve the root** with `git rev-parse --show-toplevel`; never navigate above it
- **"Repo root"** in any skill instruction means the current worktree root, not the main working tree
- **Detect secondary worktrees** with `git worktree list`; if `$PWD` does not match the first entry's path, you are in a secondary worktree and must stay within it
- **Changes committed** in a secondary worktree are isolated to that worktree's branch — do not stage or commit files from outside it

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
