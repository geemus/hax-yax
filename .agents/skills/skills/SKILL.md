---
name: skills
description: >
  Creates, updates, and manages reusable agent skills.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Skills

Manages the lifecycle of reusable skills stored under `.agents/skills/`.

## Instructions

### Creating a skill

1. Choose a name: lowercase letters, numbers, and hyphens only; 1–64 chars; must be unique under `.agents/skills/`
2. Create the directory: `.agents/skills/<skill-name>/`
3. Create `SKILL.md` — required frontmatter fields: `name` (must match directory), `description` (what it does and when to use it)
4. Write a clear Markdown body with instructions agents can follow directly
5. Add `scripts/`, `references/`, or `assets/` subdirectories only when needed for Level 3 content
6. Review `AGENTS.md` — update if the new skill affects documented structure, conventions, or workflow
7. Commit: `add <skill-name> skill: <one-line summary>`

### Updating a skill

1. Read the existing `SKILL.md` before making changes
2. Keep the `name` field in sync with the directory name — never rename one without the other
3. Update `metadata.version` when the instructions change meaningfully
4. Review `AGENTS.md` — update if the changes affect anything documented there
5. Commit: `update <skill-name> skill: <one-line summary of change>`

### Deleting a skill

1. Confirm with the user before deleting
2. Remove the entire `.agents/skills/<skill-name>/` directory
3. Review `AGENTS.md` — remove any references to the deleted skill
4. Commit: `remove <skill-name> skill: <reason>`

### Reviewing / auditing skills

- List all skills: `ls .agents/skills/`
- Validate a skill: check that `SKILL.md` exists, frontmatter has `name` and `description`, `name` matches the directory
- For full format rules, see `references/skill-format.md`
- If the audit is read-only and no issues are found, no commit is needed
- If audit findings require corrections, apply fixes and commit: `fix <skill-name> skill: <description of issue corrected>`

## Progressive Disclosure

| Level | Content | When to load |
|-------|---------|--------------|
| 1 | `name` + `description` | Always |
| 2 | `SKILL.md` body (this file) | When skill is triggered |
| 3 | `scripts/`, `references/`, `assets/` | When you need format details, examples, or executables |

Read `references/skill-format.md` for complete frontmatter rules, naming constraints, and a full worked example.
