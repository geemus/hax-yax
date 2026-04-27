# hax-yax

Reusable agent skills for Claude Code and other AI coding agents. Follows the [agentskills.io](https://agentskills.io) open standard and ships a native [Claude Code plugin manifest](https://docs.claude.ai/code/plugins) for one-command installation.

## Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `audit-skills` | `/hax-yax:audit-skills` | Produce a full health report on the skill library: structural validation per skill and cross-skill analysis covering trigger conflicts, redundancy, workflow gaps, and composition chains |
| `create-commit` | `/hax-yax:commit` | Stage changes safely, generate a conventional-commit message, and block secrets from being committed |
| `format-review-comments` | applied proactively during code review | Format review and feedback comments using the Conventional Comments standard |
| `implement-plan` | `/hax-yax:implement-plan` | Fetch a structured plan from a GitHub issue and orchestrate the full delivery workflow: step through unchecked tasks in dependency order, commit, review, validate, and open a PR |
| `refine-prose` | `/hax-yax:refine-prose` | Polish drafted prose for clarity, conciseness, and consistent voice before presenting or posting |
| `review-agents-md` | `/hax-yax:review-agents-md` | Audit AGENTS.md and equivalent agent instruction files for completeness, accuracy, agent-friendliness, freshness, and scope coherence |
| `review-plan` | `/hax-yax:review-plan` | Evaluate a structured work plan across six quality dimensions: objective clarity, task actionability, dependency correctness, acceptance criteria completeness, scope appropriateness, and structural completeness |
| `review-pr` | `/hax-yax:review-pr` | Conduct a systematic PR review across logic, security, performance, test coverage, and documentation |
| `review-skill` | `/hax-yax:review-skill` | Evaluate a skill for effectiveness across instruction clarity, trigger discoverability, example completeness, composability, and scope coherence |
| `upsert-agents-md` | `/hax-yax:upsert-agents-md` | Create or update AGENTS.md and equivalent agent instruction files by surveying the repository |
| `upsert-plan` | `/hax-yax:upsert-plan` | Create and update structured work plans as GitHub issues |
| `upsert-skill` | `/hax-yax:upsert-skill` | Manage the lifecycle of reusable agent skills stored under `.agents/skills/` |

## Installation

### Claude plugin (recommended)

Install from a local clone using `--plugin-dir`:

```sh
git clone https://github.com/geemus/hax-yax
claude --plugin-dir ./hax-yax
```

Or install directly from the repository URL once Claude Code supports remote plugin installation:

```sh
claude plugin install https://github.com/geemus/hax-yax
```

Skills are registered under the `hax-yax` namespace (e.g. `/hax-yax:upsert-plan`). Run `/help` inside Claude Code to see all available commands.

### Standalone Claude Code use

If you already have the repository checked out and want skills available without the plugin namespace, the `.claude/skills` symlink points to `.agents/skills/` for direct discovery by Claude Code:

```sh
# From within the repository, skills load automatically.
# From another project, symlink or copy the skills directory:
ln -s /path/to/hax-yax/.agents/skills .claude/skills
```

## Repository structure

```
├── AGENTS.md                  # Agent guidance (agents.md standard)
├── CHANGELOG.md               # Version history
├── LICENSE
├── README.md                  # This file
├── .claude/
│   └── skills -> ../.agents/skills   # Symlink for standalone Claude Code use
├── .claude-plugin/
│   └── plugin.json            # Claude Code plugin manifest
└── .agents/
    └── skills/
        └── <skill-name>/      # One directory per skill
            ├── SKILL.md       # Required: YAML frontmatter + instructions
            ├── scripts/       # Optional: executable scripts
            ├── references/    # Optional: reference materials
            └── assets/        # Optional: templates and examples
```

## Contributing

Skills follow the format defined in `.agents/skills/upsert-skill/references/skill-format.md`. Use the `upsert-skill` skill (`/hax-yax:upsert-skill`) to create or update skills with guided steps and validation.

Commits follow [Conventional Commits](https://www.conventionalcommits.org): `<type>[scope]: <description>`.

## License

MIT — see [LICENSE](LICENSE).
