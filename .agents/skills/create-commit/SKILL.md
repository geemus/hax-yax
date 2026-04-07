---
name: create-commit
description: >
  Stages changes safely, generates a conventional-commit message from the diff,
  and blocks secrets or debug artifacts from being committed.
  TRIGGER when: user asks to commit, save changes, stage files, write a commit
  message, or run git commit.
  DO NOT TRIGGER when: user asks to push, open a PR, or create a branch.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.2"
---

# Create Commit

Guides agents through safe, well-formed git commits. Stages files automatically (skipping secrets and artifacts), generates a conventional-commit message from the diff, and confirms before committing.

## Instructions

### 1. Survey changes

```
git status
git diff
```

Identify:
- Modified, added, and deleted tracked files
- Untracked files the user likely intends to include
- Files to **exclude**: binaries, build artifacts, generated files (e.g. `dist/`, `*.pyc`, `node_modules/`, `*.min.js`), lock files unless they changed intentionally

### 2. Detect secrets

Before staging anything, scan all candidate files for secret patterns. Read `references/secret-patterns.md` for the full pattern list.

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

### 5. Confirm and commit

Present the proposed commit message to the user and ask for approval or edits.

Once approved, commit using a HEREDOC to avoid shell quoting issues:

```bash
git commit -m "$(cat <<'EOF'
<type>[scope]: <subject>

<body if any>
EOF
)"
```

## Examples

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

