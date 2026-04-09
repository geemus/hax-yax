---
name: audit-skills
description: >
  Produces a complete health report on a skill library: per-skill structural
  validation followed by cross-skill analysis covering trigger conflicts,
  redundancy, workflow coverage gaps, and composition chains. Auto-fixes
  unambiguous minor structural issues (whitespace, missing optional metadata)
  and formats all other findings with format-review-comments.
  TRIGGER when: user asks to audit skills, check skill coverage, validate all
  skills, find gaps in skills, check for trigger conflicts, or health-check the
  skill library.
  DO NOT TRIGGER when: user asks to create, update, rename, or delete a skill
  (use upsert-skill instead); or to evaluate a single skill for quality
  (use review-skill instead).
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Audit Skills

Produces a two-pass health report on the entire skill library: structural validation per skill, then cross-skill analysis covering trigger conflicts, redundancy, workflow gaps, and composition chains. All findings are formatted with `format-review-comments`. Minor issues are auto-fixed silently; everything else is reported.

## Instructions

### Setup

Read `references/checks.md` before beginning. It contains the authoritative check specifications, coverage map, and output format details referenced throughout these instructions.

### Pass 1 — Structural Validation

Run structural checks on every skill. Discover all skills with:

```
Glob: .agents/skills/*/SKILL.md
```

For each skill found, perform the checks listed in `references/checks.md` under **Structural Checks**. Checks fall into two categories:

**Auto-fix silently** (do not report, apply the fix and move on):
- Leading or trailing whitespace in frontmatter string values
- Missing optional metadata fields (`license`, `metadata.author`, `metadata.version`)
- Minor formatting inconsistencies in the SKILL.md body (list indentation, blank lines)

**Report as findings** (everything else):
- `name` field absent or empty
- `description` field absent or empty
- `name` value does not match the parent directory name exactly
- `description` exceeds 1024 characters
- `description` does not open with an action verb in third person
- `description` does not include a "when to invoke" condition (`Use when`, `TRIGGER when`, or equivalent)
- Skill scope overlaps with another skill but lacks `TRIGGER when` / `DO NOT TRIGGER when` disambiguation lines
- Files in `scripts/`, `references/`, or `assets/` that are not referenced anywhere in `SKILL.md`

If auto-fixes are applied, use the `create-commit` skill to stage and commit them before the final report. When fixes span multiple skills, use a single commit:

```
fix(<skill-name>): correct minor structural issues
```

(or `fix: correct minor structural issues across N skills` when fixes span multiple skills)

If no auto-fixes are needed, skip the intermediate commit.

### Pass 2 — Cross-Skill Analysis

After structural validation, run the cross-skill checks listed in `references/checks.md` under **Cross-Skill Checks**. These require reading all SKILL.md files together:

1. **Trigger conflicts** — Collect all natural-language trigger phrases from every skill's `description`. Flag any two skills that share the same phrase or where one phrase is a substring of another.

2. **Redundancy** — Compare skill scopes. Flag pairs where the stated purpose substantially overlaps (merge candidates). Use the `description` fields as the primary signal.

3. **Workflow coverage** — Map each skill to the workflow stages defined in `references/checks.md` (plan → implement → commit → review → release). Flag stages with no skill coverage.

4. **Composition gaps** — Scan all `SKILL.md` bodies for references to other skills (e.g., "use the `foo` skill", "apply `bar`"). For each reference, verify that the named skill exists under `.agents/skills/`. Flag missing skills. Also identify two-step workflows referenced across multiple skills (e.g., implement → commit, commit → review) and check whether any skill explicitly coordinates both steps. If no coordinating skill exists and the chain appears in more than one skill's instructions, flag it as a `suggestion`.

5. **Naming consistency** — Check that every skill name follows the action+object formula (verb+noun, e.g., `upsert-skill`, `create-commit`). If the library has 10 or more skills, check whether related skills share a namespace prefix. Flag deviations.

### Report Structure

Present findings in this order, as specified in `references/checks.md` under **Report Format**:

1. **Structural Findings** — one sub-section per skill with issues; skills that pass cleanly are omitted
2. **Cross-Skill Findings** — one sub-section per check category; categories with no findings are omitted
3. **Coverage Map** — rendered table showing workflow stages vs. skills (see `references/checks.md` for format)
4. **Overall Summary** — counts: skills audited, auto-fixes applied, findings by severity

Format every reported finding with the `format-review-comments` skill. Use severity labels: `issue [blocking]` for structural violations that will break agent behavior, `suggestion` for quality improvements, `nitpick` for style issues.

Apply the `refine-prose` skill to the full report before presenting it. Do not announce this step.

## Examples

**Invocation:**
```
audit my skills
```
or
```
check skill coverage
```
or
```
validate all skills
```

**Sample coverage map table:**

| Stage     | Skills                                   | Status   |
|-----------|------------------------------------------|----------|
| plan      | upsert-plan                              | covered  |
| implement | upsert-skill, review-skill, audit-skills | covered  |
| commit    | create-commit                            | covered  |
| review    | review-pr, review-skill                  | covered  |
| release   | —                                        | **gap**  |

**Sample trigger-conflict finding:**

```
suggestion: upsert-skill and audit-skills both trigger on "audit skills"

Both skills list "audit skills" as a trigger phrase. Add DO NOT TRIGGER
disambiguation to clarify that upsert-skill handles structural audits as
part of lifecycle management while audit-skills produces a full library
health report.
```

**Sample structural finding:**

```
issue [blocking](review-skill): name field does not match directory

The `name` frontmatter field reads "review_skill" but the directory is
named "review-skill". Rename the frontmatter value to "review-skill".
```
