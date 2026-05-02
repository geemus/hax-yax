---
name: upsert-agents-md
description: >
  Creates or updates agent instruction files (AGENTS.md, CLAUDE.md) by surveying
  the repository and generating accurate, current, agent-friendly guidance. Use
  when the user wants to generate, create, write, update, refresh, or regenerate
  an AGENTS.md or CLAUDE.md for a repository.
  TRIGGER when: user asks to create, generate, write, update, or refresh an
  AGENTS.md, CLAUDE.md, or agent instruction file.
  DO NOT TRIGGER when: user asks to review, audit, check, or evaluate an existing
  agent instruction file (use review-agents-md instead).
license: Apache-2.0
metadata:
  author: geemus
  version: "1.2"
---

# Upsert AGENTS.md

Creates or updates agent instruction files by surveying the actual repository and generating accurate, agent-friendly guidance.

## Instructions

### 1. Determine mode and target file

**Mode** — if the target file already exists, this is **update mode**; otherwise **create mode**.

**Target file** — if the user provides a filename argument (e.g. `/upsert-agents-md CLAUDE.md`), use that path. Otherwise, search the repository for existing agent instruction files:

- `AGENTS.md` (worktree root)
- `CLAUDE.md` (worktree root or `.claude/CLAUDE.md`)
- `CURSOR.md` (worktree root)
- `.github/copilot-instructions.md`
- `.cursorrules`

Use `Glob` to locate all matches.

- **Exactly one file found:** use it as the target (update mode).
- **Multiple files found:** list each path and ask the user which to update.
- **No files found:** default to `AGENTS.md` at the worktree root (create mode).

In **update mode**, read the full contents of the existing file before continuing. Note any intentional customizations (non-standard sections, hand-written prose) to preserve them.

### 2. Survey the repository

Gather the information needed to write accurate guidance. Work through each area below.

**Repository purpose and metadata**
- Read `README.md` if it exists — extract a one-line purpose statement and any key domain details
- Check `package.json`, `pyproject.toml`, `Cargo.toml`, or equivalent for project name, version, and language

**Directory structure**
- Run `Glob` with `**/*` (or equivalent) limited to depth 2–3 to produce a representative layout
- Identify key directories: source, tests, scripts, config, docs
- Note any standard conventions (e.g. `src/`, `.agents/skills/`, `.github/`)

**Skills** *(if `.agents/skills/` exists)*
- Glob `.agents/skills/*/SKILL.md`
- For each skill, read the frontmatter `name` and first sentence of `description`
- Build the skills table from this data — do not invent entries

**Git workflow**
- Run `git remote get-url origin` to confirm the remote
- Run `git branch -a` to identify the default branch and any documented branch naming conventions
- Check for `CHANGELOG.md`, `.github/`, or commit-lint config

**Tooling and environment**
- Check for `package.json` scripts, `Makefile`, shell scripts in `scripts/` or `bin/`
- Identify CLI tools the agent must have: note only tools actually referenced in the repo
- Check `.env.example` or docs for required environment variables

**Security constraints**
- Note any secrets-scanning config (`.gitleaks.toml`, `.pre-commit-config.yaml`)
- Check for destructive scripts that require confirmation

**Coding conventions**
- Check for `.eslintrc`, `.prettierrc`, `ruff.toml`, `.editorconfig`, or style docs
- Note language(s), formatting rules, and test commands

### 3. Draft the agent instruction file

Produce a Markdown document with these sections. Include only sections for which you found actual content — do not add placeholder sections. In **update mode**, preserve any existing section that is not stale; replace or augment sections where the survey found newer information.

**Where information belongs.** Before writing each section, decide whether the content belongs in the agent instruction file at all. Use this heuristic:

| Content type | Home |
|---|---|
| How a module works, callback contracts, struct fields, type signatures | In-source documentation (docstrings, doc comments, header blocks) |
| User-facing setup, installation, and workflow | `README.md` |
| Agent decision rules — what to prefer, what to avoid, invariants to uphold | `AGENTS.md` (this file) |

For every paragraph or table you are about to write, ask: *could a future agent find this by reading the relevant code or `README.md`?* If yes, emit a one-line pointer (e.g. *"See `Retcon.Message` for the atom/string key contract."*) instead of repeating the prose. If no, emit the full explanation — it has no other home.

**Do not duplicate in-source documentation in the agent instruction file.** Implementation details belong next to the code (in docstrings, doc comments, or whatever each language uses); agent decision rules belong here. Duplicated prose creates two places to keep in sync on every refactor and crowds out the decision rules that justify this file's existence.

#### Repository Purpose *(required)*
One short paragraph stating what the repository does and who it is for. Write for an agent, not a human reader.

#### Structure *(required)*
A fenced code block with an annotated directory tree. Include only paths that actually exist. Annotate each entry with a brief purpose comment.

#### Available Skills *(include only if `.agents/skills/` exists)*
A Markdown table with columns `Skill`, `Trigger`, and `Purpose`. Populate from the survey; do not invent rows.

#### Git Workflow *(required)*
Cover: default branch name, feature branch naming convention, commit message format (with examples), and PR process. Be explicit — state exact formats, not vague guidance.

#### Code Style & Conventions *(include if conventions were found)*
List formatting rules, language-specific style constraints, and test commands. Use imperative phrasing agents can follow directly.

#### Security *(required)*
Document: secrets management rules, any off-limits destructive operations, and confirmation requirements for destructive scripts. If no constraints were found beyond standard hygiene, include a minimal section noting "Never commit secrets or credentials."

#### Environment *(include if tools or env vars are required)*
List required CLI tools and environment variables. For each, note what it is used for and where to obtain it.

### 4. Leanness pass

Before self-reviewing, walk the draft section-by-section and prune duplicated prose. For each paragraph, table, or list:

1. **Locate the canonical home.** For module behavior, callback contracts, struct fields, type signatures, or function semantics, open the relevant source file and check its in-source documentation — whichever convention the language uses (docstrings, doc comments, top-of-file headers, or equivalent). For setup, installation, or workflow narrative, check `README.md`.
2. **If the canonical home already covers it:** replace the section's prose with a one-line pointer naming the file or module — e.g. *"See `Retcon.Message` for the atom/string key contract."* or *"See `README.md` § Installation for setup."* Do not also restate the content.
3. **If the canonical home is missing or thin:** the agent instruction file is not the right place to fix that. Note the gap to the user when presenting the draft (step 6) so they can update the code docs or README, but keep this file lean.
4. **Keep what only belongs here:** agent decision rules, preferences, invariants, branch and commit conventions, and explicit do/do-not guidance — these have no other canonical home.

After this pass, every retained paragraph should answer *"what should an agent prefer, avoid, or uphold?"* rather than *"how does this module work?"*

### 5. Self-review

Before presenting, check the draft against the `review-agents-md` checklist dimensions silently:

- **Completeness** — required sections present?
- **Accuracy** — all paths, branch names, and tool names verified against the actual repo?
- **Agent-friendliness** — instructions are explicit and imperative, not vague?
- **Freshness** — no stale references; skills table matches actual skill directories?
- **Scope coherence** — focused on agent guidance, no README/CHANGELOG duplication?
- **Leanness** — no section mirrors in-source documentation verbatim; duplicated prose has been replaced with pointers?

Fix any issues found before presenting.

### 6. Present the draft

State whether this is **create mode** (new file) or **update mode** (file path will be updated). Show the full draft and ask for confirmation before writing.

In **update mode**, also briefly describe what changed relative to the existing file: sections added, revised, or preserved.

### 7. Write the file

After confirmation, write the file to the target path. In **update mode**, overwrite the existing file entirely with the approved content.

### 8. Validate with review-agents-md

Run the `review-agents-md` skill on the file just written. If it surfaces any `issue [blocking]` findings, address them immediately and re-write the file. Surface `suggestion` findings to the user as follow-up opportunities; do not block.

### 9. Commit

Commit with an appropriate message:
- **Create mode:** `docs: add AGENTS.md`
- **Update mode:** `docs: update AGENTS.md` *(or the target filename)*

## Examples

**Create AGENTS.md from scratch:**
```
/upsert-agents-md
```
Finds no existing agent instruction file, surveys the repo, generates `AGENTS.md`, presents it for review, writes it, validates, and commits.

**Update an existing CLAUDE.md:**
```
/upsert-agents-md CLAUDE.md
```
Reads the existing `CLAUDE.md`, surveys the repo for changes, produces an updated draft, presents the diff summary, writes on confirmation, and validates.

**Auto-detect and update:**
```
/upsert-agents-md
```
Finds `AGENTS.md` in the worktree root, enters update mode automatically.
