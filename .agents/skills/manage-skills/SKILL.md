---
name: manage-skills
description: >
  Manages the lifecycle of reusable agent skills stored under .agents/skills/.
  Use when the user wants to add, modify, rename, delete, or audit skills in
  this repository — also triggered by "create a new skill", "set up a skill",
  "list my skills", "what skills do I have", or "remove a skill".
  DO NOT TRIGGER when: the user asks to review, evaluate, or critique a skill
  for quality (use review-skill instead).
license: Apache-2.0
metadata:
  author: geemus
  version: "1.6"
---

# Manage Skills

Manages the lifecycle of reusable skills stored under `.agents/skills/`.

## Instructions

### Creating a skill

1. Choose a name — it must pass all of the following checks:
   - **Format**: lowercase letters, digits, and hyphens only; 1–64 chars; no leading, trailing, or consecutive hyphens; must be unique under `.agents/skills/`
   - **Quality**: use an action+object formula (`search-orders`, `extract-pdf-text`); avoid single-word or noun-only names; name the outcome, not the implementation; avoid reserved words (`anthropic`, `claude`)
   - **Namespace**: if the repository already has 10 or more skills, add a shared prefix for related skills (e.g. `orders-search`, `orders-refund`)
   - For the full naming specification and worked example, read `references/skill-format.md`
2. Create the directory: `.agents/skills/<skill-name>/`
3. Create `SKILL.md` — required frontmatter fields: `name` (must match directory), `description` (what it does and when to use it); use the **"Does X. Use when Y."** template — start with the action verb in third person, then state the triggering condition; for full specification and worked example, read `references/skill-format.md`
4. Document natural-language trigger phrases in the `description` — include the phrases users are likely to say (e.g. *"create a commit"*, *"review this PR"*, *"make a plan"*). Use the inline `Use when...` form for simple skills; add explicit `TRIGGER when:` / `DO NOT TRIGGER when:` lines when the skill overlaps with others and disambiguation is needed. See the Natural Language Triggers section in `references/skill-format.md` for both patterns and concrete examples.
5. Omit `allowed-tools` unless the user explicitly requests tool restrictions
6. Write a clear Markdown body with instructions agents can follow directly
7. Add `scripts/`, `references/`, or `assets/` subdirectories only for content too large to inline in `SKILL.md` (large examples, schemas, scripts, reference docs). See `references/skill-format.md` for the full progressive disclosure model.
8. Apply the `refine-prose` skill to the `description` frontmatter field and the `SKILL.md` body — run it silently before saving
9. Review `AGENTS.md` — update if the new skill affects documented structure, conventions, or workflow
10. Commit: `feat(<skill-name>): add <skill-name> skill — <one-line summary>`

### Updating a skill

1. Read the existing `SKILL.md` before making changes
2. Keep the `name` field in sync with the directory name — never rename one without the other
3. Update `metadata.version` when the instructions change meaningfully
4. If touching the `description`, review the natural-language trigger phrases — ensure the `Use when...` condition and any `TRIGGER when:` / `DO NOT TRIGGER when:` lines reflect the updated skill scope. See the Natural Language Triggers section in `references/skill-format.md` for guidance.
5. Apply the `refine-prose` skill to any `description` or body text that is rewritten or newly authored — run it silently before saving.
6. Review `AGENTS.md` — update if the changes affect anything documented there
7. Commit: `feat(<skill-name>):` or `fix(<skill-name>):` or `docs(<skill-name>):` depending on the nature of the change, followed by a lowercase imperative description

### Deleting a skill

1. Confirm with the user before deleting
2. Remove the entire `.agents/skills/<skill-name>/` directory
3. Review `AGENTS.md` — remove any references to the deleted skill
4. Commit: `chore(<skill-name>): remove <skill-name> skill — <reason>`

### Reviewing / auditing skills

1. Discover all skills: use `Glob` with pattern `.agents/skills/*/SKILL.md`
2. For each skill found, validate:
   - `SKILL.md` exists (confirmed by the glob result)
   - Frontmatter contains `name` and non-empty `description`
   - `name` value matches the parent directory name exactly
   - `description` describes both what the skill does and when to invoke it
3. For complete format rules and validation criteria, read `references/skill-format.md`
4. For quality evaluation of individual skills (instruction clarity, trigger discoverability, composability), use the `review-skill` skill
5. Auto-fix minor issues (typos, table formatting, missing optional metadata fields); ask the user before removing or renaming skills
6. Apply the `refine-prose` skill to any `description` or body text that is rewritten or newly authored during the audit — run it silently before saving
7. If the audit is read-only and no issues are found, no commit is needed
8. If audit findings require corrections, apply fixes and commit: `fix(<skill-name>): <description of issue corrected>`

## Examples

**Creating a new skill called `deploy`:**
> "Create a skill that deploys the app to staging using the deploy.sh script."

Expected: agent creates `.agents/skills/deploy/SKILL.md` with appropriate frontmatter and instructions, commits with `feat(deploy): add deploy skill — ...`

**Updating an existing skill's description:**
> "Update the skills skill description to mention it also handles auditing."

Expected: agent reads existing `SKILL.md`, updates the `description` frontmatter, bumps `metadata.version`, commits with `docs(skills): update description to mention auditing`

**Deleting an obsolete skill:**
> "Remove the old `pdf` skill, we don't use it anymore."

Expected: agent confirms with user, removes `.agents/skills/pdf/` directory, updates `AGENTS.md` if needed, commits with `chore(pdf): remove pdf skill — no longer used`

**Auditing all skills:**
> "List all my skills and check if any have issues."

Expected: agent globs `.agents/skills/*/SKILL.md`, validates each skill's frontmatter and name/directory alignment, reports findings, auto-fixes minor issues (e.g. typos, table formatting), commits any corrections with `fix(<skill-name>): <description of issue corrected>`

