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

#### Semantic quality rules

- **Action + object formula**: prefer `search-orders`, `analyze-sentiment`, `generate-report` over vague single words
- **Gerund form acceptable**: `processing-pdfs`, `extracting-metadata` are valid when they read naturally
- **Specificity over brevity**: single-word names are almost always too vague — `pdf` says nothing about what happens to the PDF
- **Name for outcome, not implementation**: use `analyze-sentiment` not `run-bert-v2`; callers should not need to know the underlying tool or model
- **Avoid reserved words**: do not use `anthropic`, `claude`, or other trademark terms as the skill name or prefix
- **Namespace prefixes for large repos**: when a repository has 10 or more skills, prefix related skills with a shared namespace (e.g. `orders-search`, `orders-refund`) to make discovery easier and prevent name collisions

### `description` rules

- Non-empty; maximum 1024 characters
- Written in third person (agents read this for discovery)
- Must describe both *what* the skill does and *when* to invoke it

#### Description best practices

- **"Does X. Use when Y." template**: open with what the skill does, then explicitly state the triggering condition. Example: *"Extracts text from PDF files. Use when the user asks to read, parse, or summarize a PDF document."*
- **Third person, active voice, start with the action verb**: write *"Generates …"* not *"This skill will generate …"* or *"Can be used to generate …"*
- **Include trigger keywords and synonyms**: if users might phrase the request multiple ways, mention the synonyms so agent discovery works reliably (e.g. *"… also triggered by requests to OCR or convert PDFs"*)
- **Explicitly distinguish overlapping skills**: if two skills handle similar inputs, call out the difference in each description so the agent picks the right one

#### Natural Language Triggers

The `description` field is the primary discovery mechanism — agents match it against user phrasing at session startup. Trigger phrases are the natural-language expressions users actually say that should activate the skill.

**Two patterns for documenting triggers:**

1. **`Use when...` (inline)** — embed the triggering condition directly in the description. Best for simple, unambiguous skills:
   > *"Generates a structured work plan. Use when the user asks to create a plan, define tasks, or scope out work."*

2. **`TRIGGER when` / `DO NOT TRIGGER when` (explicit block)** — add structured lines after the description for skills that overlap with others or need clear disambiguation:
   > *"Stages changes and creates a conventional commit message.*
   > *TRIGGER when: user asks to commit, stage files, write a commit message, or save changes.*
   > *DO NOT TRIGGER when: user asks to push, open a PR, or create a branch."*

**Concrete examples:**

| User says | Skill triggered | Key phrases in description |
|-----------|----------------|---------------------------|
| "create a commit", "save my changes", "commit this" | `create-commit` | *commit, stage files, write a commit message* |
| "review this PR", "can you audit this diff" | `review-pr` | *pull request review, audit a diff* |
| "make a plan", "create plan skill", "scope out this work" | `upsert-plan` | *work plan, scope out, define tasks* |
| "clean up this draft", "polish my writing" | `refine-prose` | *polish prose, clarity, written output* |

**Tips:**
- Include synonyms users might say: *"commit"* and *"save changes"* should both trigger `create-commit`
- For overlapping skills, `DO NOT TRIGGER when` prevents the wrong skill from firing
- Natural phrasing (*"can you review this PR"*) should match as reliably as a direct slash command (*"/review-pr"*)

### `allowed-tools` rules

Omit unless the user explicitly requests tool restrictions. Restricting tools limits the agent's ability to handle unexpected situations; only add this field when there is a specific reason to do so.

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

**Do not** add a progressive disclosure table to individual `SKILL.md` files. The table above describes how the system works; copying it into each skill is redundant and should be removed when found.

## Worked Example

Directory: `.agents/skills/extract-pdf-text/`

```
extract-pdf-text/
├── SKILL.md
└── references/
    └── pdfminer-api.md
```

`SKILL.md`:

```markdown
---
name: extract-pdf-text
description: >
  Extracts and summarizes text content from PDF files. Use when the user
  asks to read, parse, extract, or summarize text from a PDF document.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Extract PDF Text

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
- **Plugin self-containment**: each `SKILL.md` must be independently usable when the skill is installed as a plugin in another repository — do not rely on this repository's `AGENTS.md` or any other repo-level guidance being available to the agent

### Anti-patterns

Avoid these common mistakes when naming and describing skills:

| Anti-pattern | Example | Problem | Better alternative |
|---|---|---|---|
| Generic names | `helper`, `utils`, `tools` | Conveys no useful information for discovery | Name what the skill specifically does: `format-date`, `validate-email` |
| Noun-only names | `data`, `files`, `report` | Does not indicate what action is taken | Add the verb: `fetch-data`, `archive-files`, `generate-report` |
| Implementation-detail names | `run-bert-v2`, `call-gpt4` | Breaks callers when the implementation changes | Name the outcome: `analyze-sentiment`, `summarize-text` |
| Mixed casing | `SearchOrders`, `search_orders` | Violates the lowercase-kebab-case format rule | `search-orders` |
| Vague description openers | *"This skill can be used to…"*, *"A helper that…"* | Wastes the first tokens agents read; passive and weak | Start with the action verb: *"Searches orders by…"* |
