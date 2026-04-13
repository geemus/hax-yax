---
name: review-agents-md
description: >
  Audits AGENTS.md files and equivalent agent instruction files (CLAUDE.md,
  CURSOR.md, Copilot instructions) for completeness, accuracy, and
  agent-friendliness, producing structured, actionable feedback. Cross-references
  file paths, scripts, and branch names against the actual repository to catch
  stale or incorrect guidance. Formats every finding with format-review-comments
  and runs a refine-prose pass before presenting results.
  TRIGGER when: user asks to review, audit, check, inspect, analyze, or
  evaluate an AGENTS.md, CLAUDE.md, or agent instruction file.
  DO NOT TRIGGER when: user asks to review a pull request (use review-pr
  instead) or to evaluate a skill file (use review-skill instead).
license: Apache-2.0
metadata:
  author: geemus
  version: "1.1"
---

# Review AGENTS.md

Audits agent instruction files for completeness, accuracy, and agent-friendliness across five dimensions, with cross-referencing checks to verify that paths, scripts, and branch names mentioned in the file actually exist in the repository.

## Instructions

### 1. Detect the target file

If the user provides a file path or filename as an argument (e.g., `/review-agents-md CLAUDE.md`), use that file directly and skip to Step 2. If the file does not exist, stop and report: "File `<path>` not found. Provide a valid path to an agent instruction file."

Otherwise, search the repository for agent instruction files. Look for any of the following:

- `AGENTS.md` (worktree root)
- `CLAUDE.md` (worktree root or `.claude/CLAUDE.md`)
- `CURSOR.md` (worktree root)
- `.github/copilot-instructions.md`
- `.cursorrules`

Use `Glob` or `Bash` to locate all matches.

- **Exactly one file found:** proceed with that file.
- **Multiple files found:** list each path and ask the user: "I found the following agent instruction files. Which would you like me to review?" Wait for confirmation before continuing.
- **No files found:** stop and tell the user: "No agent instruction file found in this repository. Expected AGENTS.md, CLAUDE.md, CURSOR.md, or .github/copilot-instructions.md."

### 2. Read the target file

Read the full contents of the chosen file before reviewing. Do not begin reviewing until the file has been read in full.

### 3. Review each dimension

Read `assets/review-checklist.md` before starting. Work through every dimension in order, applying each checklist item and noting findings as you go. Record whether an inline fix is clear and unambiguous for each finding.

The five dimensions are:

#### Completeness

Does the file cover all essential topics an agent needs to operate correctly? Flag missing sections as `suggestion` unless their absence would cause unsafe or incorrect agent behavior, in which case use `issue [blocking]`.

#### Accuracy (cross-referencing)

Do all references in the file point to things that actually exist? Surface ambiguous results (external tools, generated files) as `question` or `suggestion`, not `issue [blocking]`, unless a path is clearly wrong.

#### Agent-friendliness

Can an agent follow the instructions without human interpretation? Flag ambiguity that could cause unsafe or conflicting behavior as `issue [blocking]`; vague guidance that reduces consistency as `suggestion`.

#### Freshness

Does the documented state match the actual repository? Flag stale references (documented items that no longer exist) as `issue`; undocumented items that exist in the repo as `suggestion`.

#### Scope coherence

Is the file focused on agent guidance, or does it duplicate README/CHANGELOG content or serve a mixed audience? Flag content that actively misleads an agent as `issue [blocking]`; content that belongs elsewhere as `suggestion`.

### 4. Format all findings with format-review-comments

Apply the `format-review-comments` skill to every comment. Choose the label that matches the severity and intent:

| Situation | Recommended label |
|-----------|------------------|
| Causes incorrect or unsafe agent behavior | `issue [blocking]` |
| Strong improvement | `suggestion` |
| Minor style or wording | `nitpick` |
| Ambiguous reference that may or may not be wrong | `question` |
| Small unambiguous fix | `todo` |
| Informational only | `note` |

Do not write `praise` comments. Omit positive findings entirely.

### 5. Suggest inline fixes for unambiguous findings

For any finding where the correct fix is clear — a rewritten sentence, a corrected path, an added section — include the suggested replacement inline with the comment so the reviewer can apply it directly. Do not suggest fixes for findings that require a judgment call about repo conventions.

### 6. Write a review summary

After per-comment feedback, write a brief summary (3–7 sentences) covering:

- Overall assessment — effective as-is, needs minor improvements, or needs significant rework
- The most important finding (name any blocking issues)
- Any patterns visible across the dimensions
- Any skipped dimensions and why

Format the summary as a `note` conventional comment.

### 7. Refine prose

Apply the `refine-prose` skill to all output before presenting it. Do not announce this step to the user.

## Examples

**Auto-detection, single file:**
```
/review-agents-md
```
Finds `AGENTS.md` in the worktree root and proceeds automatically.

**Sample output — no agent instruction file found:**
```
No agent instruction file found in this repository. Expected AGENTS.md, CLAUDE.md, CURSOR.md, or .github/copilot-instructions.md.
```

**Explicit target:**
```
/review-agents-md CLAUDE.md
```

**Multiple files detected:**
```
I found the following agent instruction files:
- AGENTS.md
- CLAUDE.md

Which would you like me to review?
```

**Sample output — accuracy finding (missing path):**
```
issue: `.agents/scripts/deploy.sh` is referenced in the Git Workflow section but does not exist in the repository

Remove the reference or add the missing script:
> Remove: "Run `.agents/scripts/deploy.sh` before merging"
```

**Sample output — accuracy finding (ambiguous tool):**
```
question: the file references the `sprite` CLI in the Tooling section — this tool was not found locally. Is `sprite` expected to be installed separately, or is this reference stale?
```

**Sample output — agent-friendliness finding:**
```
suggestion: "follow good commit hygiene" in the Git Workflow section is vague — an agent cannot determine what this means without further context

Replace with an explicit rule:
> Use Conventional Commits format: `<type>[scope]: <description>` where type is one of feat, fix, docs, refactor, test, chore.
```

**Sample output — freshness finding:**
```
issue: the Skills table lists `create-plan` but `.agents/skills/create-plan/` does not exist in the repository — this entry is stale

Remove the `create-plan` row from the table.
```

**Sample output — summary:**
```
note: overall this AGENTS.md needs minor improvements before it reliably guides agents. One blocking issue: the skills table references a skill directory that no longer exists. Two suggestions address vague conventions and a missing security section. Cross-referencing found all file paths accurate. Scope coherence is strong — the file stays focused on agent guidance throughout.
```
