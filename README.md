# skills

Reusable agent skills for Claude Code and other AI coding agents. Follows the [agentskills.io](https://agentskills.io) open standard and ships a native [Claude Code plugin manifest](https://docs.claude.ai/code/plugins) for one-command installation.

## Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `manage-skills` | `/skills:manage-skills` | Create, update, manage, and audit skills in this repository |
| `create-plan` | `/skills:create-plan` | Generate a structured work plan and write it to a GitHub issue |
| `format-review-comments` | applied proactively during code review | Format review and feedback comments using the Conventional Comments standard |
| `review-pr` | `/skills:review-pr` | Conduct a systematic PR review across logic, security, performance, test coverage, and documentation |
| `create-commit` | `/skills:commit` | Stage changes safely, generate a conventional-commit message, and block secrets from being committed |
| `refine-prose` | `/skills:refine-prose` | Polish drafted prose for clarity, conciseness, and consistent voice before presenting or posting |

## Installation

### Claude plugin (recommended)

Install from a local clone using `--plugin-dir`:

```sh
git clone https://github.com/geemus/skills
claude --plugin-dir ./skills
```

Or install directly from the repository URL once Claude Code supports remote plugin installation:

```sh
claude plugin install https://github.com/geemus/skills
```

Skills are registered under the `skills` namespace (e.g. `/skills:create-plan`). Run `/help` inside Claude Code to see all available commands.

### Standalone Claude Code use

If you already have the repository checked out and want skills available without the plugin namespace, the `.claude/skills` symlink points to `.agents/skills/` for direct discovery by Claude Code:

```sh
# From within the repository, skills load automatically.
# From another project, symlink or copy the skills directory:
ln -s /path/to/skills/.agents/skills .claude/skills
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

Skills follow the format defined in `.agents/skills/manage-skills/references/skill-format.md`. Use the `manage-skills` skill (`/skills:manage-skills`) to create or update skills with guided steps and validation.

Commits follow [Conventional Commits](https://www.conventionalcommits.org): `<type>[scope]: <description>`.

## License

MIT — see [LICENSE](LICENSE).
