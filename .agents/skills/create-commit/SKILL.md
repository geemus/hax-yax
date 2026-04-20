---
name: create-commit
description: >
  Stages changes safely, generates a conventional-commit message from the diff,
  and blocks secrets or debug artifacts from being committed.
  TRIGGER when: user asks to commit, save changes, stage files, write a commit
  message, run git commit, check in my changes, or commit my work.
  DO NOT TRIGGER when: user asks to push, open a PR, or create a branch.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.4"
---

# Create Commit

Guides agents through safe, well-formed git commits. Stages files automatically (skipping secrets and artifacts), generates a conventional-commit message from the diff, and confirms before committing.

## Instructions

**Side effects:** stages files (`git add`), creates a git commit object. Does not push or open a PR.

### 1. Survey changes

```
git status
git diff
```

If `git status` shows no changes (clean working tree), stop and tell the user: "Nothing to commit — the working tree is clean."

Identify:
- Modified, added, and deleted tracked files
- Untracked files that are clearly part of the work (source files, config, docs in the project tree)
- Files to **exclude**: binaries, build artifacts, generated files (e.g. `dist/`, `*.pyc`, `node_modules/`, `*.min.js`), lock files unless they changed intentionally

If the untracked file set is ambiguous (many files, files that look auto-generated, or files outside the project's normal structure), list them and ask the user which to include before proceeding.

### 2. Detect secrets

Before staging anything, scan all modified, added, and untracked files identified in step 1 for secret patterns. Read `references/secret-patterns.md` for the full pattern list.

If any match is found:
- **Stop** — do not stage the affected file
- Report the file path and the type of pattern matched
- Suggest remediation (remove the value, use an environment variable, add the file to `.gitignore`)
- Continue with the remaining safe files; do not abort the entire commit unless every file contains secrets

### 3. Stage automatically

Stage all safe files (those that passed step 2 and are not excluded artifacts) using individual `git add <file>` calls — never `git add .` or `git add -A`.

Report a brief summary:
- Staged: list of files
- Skipped: list of files with reason (secret detected / artifact / binary)

If nothing is staged after exclusions, stop and tell the user what was skipped and why.

### 4. Generate commit message

Read the staged diff:

```
git diff --cached
```

Derive the message following [Conventional Commits](https://www.conventionalcommits.org):

```
<type>[optional scope]: <subject>

[optional body]
```

**Type** — pick the best fit:
- `feat`: new feature or capability
- `fix`: bug fix
- `docs`: documentation only
- `refactor`: restructuring without behaviour change
- `test`: adding or updating tests
- `chore`: maintenance, tooling, dependencies

**Scope** — optional; use the skill name, component, or directory most affected (e.g. `feat(plan):`, `fix(skills):`).

**Subject** — lowercase imperative mood, no trailing period, ≤72 characters.

**Body** — include when the change is non-obvious: explain *why*, not *what*. Wrap at 72 characters.

### 5. Commit

Display the proposed commit message to the user, then commit.

## Examples

**Invocation:**
```
/commit
```

**Feature addition:**
> User has edited `.agents/skills/plan/SKILL.md` to add update mode.

```
feat(plan): add update mode for existing issues

Allows /plan to be invoked with an issue number or URL to edit an
existing issue in place rather than always creating a new one.
```

**Bug fix:**
> User fixed a typo and a broken link in `AGENTS.md`.

```
fix: correct typo and broken link in AGENTS.md
```

**Secret detected — partial commit:**
> User runs `/commit`; one file contains a hardcoded database password.

```
Staged: src/api/client.py, README.md
Skipped:
  - config/local.env  [BLOCKED: explicit secret assignment on line 4]
    Fix: remove DB_PASSWORD value; reference $DB_PASSWORD instead,
    or add config/local.env to .gitignore

Commit message:
  feat(api): add client module
```

