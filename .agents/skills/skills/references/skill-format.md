# Skill Format Reference

Full specification for the Agent Skills format used in this repository.
Canonical spec: https://agentskills.io/specification

## Directory Layout

```
.agents/skills/<skill-name>/
├── SKILL.md        # Required: YAML frontmatter + Markdown instructions
├── scripts/        # Optional: executable scripts (Python, Bash, JS, etc.)
├── references/     # Optional: reference material, API docs, schemas
└── assets/         # Optional: templates, examples, other static resources
```

## SKILL.md Frontmatter

```yaml
---
name: skill-name          # required
description: >            # required
  What the skill does and when to invoke it. Injected into agent system
  prompt — write in third person, max 1024 characters.
license: Apache-2.0       # optional
metadata:                 # optional
  author: your-handle
  version: "1.0"
allowed-tools:            # optional: restrict tool access within the skill
  - Bash
  - Read
---
```

### `name` rules

- Lowercase letters (`a–z`), digits (`0–9`), and hyphens (`-`) only
- 1–64 characters
- No leading, trailing, or consecutive hyphens
- Must match the parent directory name exactly

### `description` rules

- Non-empty; maximum 1024 characters
- Written in third person (agents read this for discovery)
- Must describe both *what* the skill does and *when* to invoke it

### `allowed-tools` rules

List tool names to restrict which tools the agent may call while executing the skill. Omit to allow all tools.

**Built-in tools** use their plain name: `Bash`, `Read`, `Edit`, `Write`, `Glob`, `Grep`, etc.

**MCP tools** follow the convention `mcp__<server>__<tool>` (double-underscore separators):

```yaml
allowed-tools:
  - Bash
  - mcp__github__create_issue
  - mcp__github__search_repositories
```

MCP tool names are runtime-dependent — they reflect whatever MCP servers are configured in the agent's environment. Document the names you require and note that the skill will fail if those MCP servers are not available.

## Markdown Body

Recommended structure:

```markdown
# Skill Name

Brief purpose statement.

## Instructions

Step-by-step guidance for the agent.

## Examples

Concrete inputs and expected outputs.
```

Write instructions agents can follow without needing to infer intent.
Prefer explicit over implicit.

## Progressive Disclosure Levels

| Level | Content | When loaded | Target size |
|-------|---------|-------------|-------------|
| 1 | `name` + `description` frontmatter | Always (startup) | ~100 tokens |
| 2 | `SKILL.md` body | When skill is triggered | < 5,000 tokens |
| 3 | `scripts/`, `references/`, `assets/` | On demand, via Bash | Unlimited |

Keep `SKILL.md` lean. Move large examples, schemas, and reference docs into `references/` or `assets/` and instruct agents to read them on demand.

## Worked Example

Directory: `.agents/skills/pdf/`

```
pdf/
├── SKILL.md
└── references/
    └── pdfminer-api.md
```

`SKILL.md`:

```markdown
---
name: pdf
description: >
  Extracts and summarizes content from PDF files. Use when the user
  asks to read, parse, or extract text from a PDF document.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
allowed-tools:
  - Bash
  - Read
---

# PDF

Extracts text and metadata from PDF files using pdfminer.

## Instructions

1. Check that `pdfminer.six` is installed: `pip show pdfminer.six`
2. Run `scripts/extract.py <path>` to extract text
3. Summarize the extracted content for the user

For full API details, read `references/pdfminer-api.md`.
```

## Conventions

- One skill per directory — do not bundle unrelated tasks
- Scripts must be self-contained or document their dependencies clearly
- Never commit secrets, tokens, or credentials inside a skill
- Skills that call external services must document required environment variables in `SKILL.md`
- Skills performing destructive operations must include an explicit confirmation step
