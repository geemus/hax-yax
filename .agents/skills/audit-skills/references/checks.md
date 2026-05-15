# Audit Checks Reference

This file defines the authoritative check specifications for the `audit-skills` skill. It is organized into three sections: structural checks (Pass 1), cross-skill checks (Pass 2), and report format.

---

## Structural Checks (Pass 1)

Run these checks on every skill discovered via `Glob: .agents/skills/*/SKILL.md`.

### Required Field Checks

| Check | Severity | Auto-fix? |
|-------|----------|-----------|
| `name` field is present in frontmatter | issue [blocking] | No |
| `name` field is non-empty | issue [blocking] | No |
| `name` value matches the parent directory name exactly | issue [blocking] | No |
| `description` field is present in frontmatter | issue [blocking] | No |
| `description` field is non-empty | issue [blocking] | No |

### Description Quality Checks

| Check | Severity | Auto-fix? |
|-------|----------|-----------|
| `description` is 1024 characters or fewer | issue [blocking] | No — report and ask user to shorten |
| `description` opens with an action verb in third person (e.g., "Produces", "Manages", "Stages") | suggestion | No |
| `description` includes a "when to invoke" condition (`Use when`, `TRIGGER when`, or equivalent phrase) | suggestion | No |
| When skill scope overlaps with another skill, `description` includes `TRIGGER when` and `DO NOT TRIGGER when` disambiguation lines | suggestion | No |

### File Reference Checks

| Check | Severity | Auto-fix? |
|-------|----------|-----------|
| Every file under `scripts/` is referenced by name somewhere in `SKILL.md` | suggestion | No |
| Every file under `references/` is referenced by name somewhere in `SKILL.md` | suggestion | No |
| Every file under `assets/` is referenced by name somewhere in `SKILL.md` | suggestion | No |

### Auto-Fix Rules

Apply the following corrections silently without reporting them. Commit fixes together in one pass before generating the final report.

- Strip leading and trailing whitespace from frontmatter string values
- Add missing optional fields with sensible defaults:
  - `license: Apache-2.0` if absent
  - `metadata: {}` block if absent (do not guess `author` or `version`)
- Normalize list indentation in SKILL.md body to 2-space indent
- Remove duplicate blank lines in SKILL.md body (collapse 3+ consecutive blank lines to 2)

---

## Cross-Skill Checks (Pass 2)

Run these checks after all skills have been read. They require comparing skills against each other.

### Trigger Conflicts

**Goal:** Identify trigger phrases that would cause an agent to ambiguously choose between two or more skills.

**Method:**
1. Extract all natural-language trigger phrases from each skill's `description` field. Phrases appear in `TRIGGER when:` lines, `Use when` clauses, and the parenthetical examples embedded in descriptions (e.g., *"also triggered by 'create a commit'"*).
2. Normalize: lowercase, strip punctuation, collapse whitespace.
3. Flag any two skills where one skill's normalized trigger phrase is an exact match or a substring of another skill's normalized trigger phrase.

**Severity:** `suggestion` for partial overlaps; `issue [blocking]` for exact matches with no disambiguation.

### Redundancy

**Goal:** Identify skills with substantially overlapping scope that are candidates for merging.

**Method:**
1. Compare every pair of skill `description` fields.
2. Flag pairs where the core action (what the skill does) is the same or where the `description` mentions the same domain without clear differentiation.
3. Do not flag skills that are intentionally layered (e.g., `review-skill` reviews one skill; `audit-skills` reviews the library — related but not redundant).

**Severity:** `suggestion`

### Workflow Coverage

**Goal:** Ensure the skill library collectively covers the full software development workflow.

**Coverage Map:**

| Stage | Description | Example skills |
|-------|-------------|----------------|
| plan | Breaking down work, defining requirements, creating issues | upsert-plan |
| implement | Writing code, creating/updating skills, making changes | upsert-skill, audit-skills |
| commit | Staging and committing changes safely | create-commit |
| review | Evaluating PRs, skills, or prose quality | review-pr, review-skill, refine-prose |
| release | Publishing, deploying, tagging versions | _(none — gap)_ |


**Method:**
1. For each skill, determine which stage(s) it primarily covers based on its `description`.
2. Render the coverage map table with the actual skills found, replacing the "Example skills" column.
3. Flag any stage with no covering skill as a gap.

**Severity:** `suggestion` for gaps

### Composition Gaps

**Goal:** Verify that skills referenced by others exist, and that chained workflows have adequate coordination.

**Method:**
1. Scan every `SKILL.md` body for skill references — patterns like:
   - `use the \`foo\` skill`
   - `apply \`bar\``
   - `invoke the \`baz\` skill`
   - `run \`qux\``
2. For each reference, check that `.agents/skills/<referenced-skill>/SKILL.md` exists. Flag missing skills as `bug`.
3. Identify sequential skill chains (skill A's output is skill B's input) and check whether a coordinating skill exists. Flag missing glue as `suggestion`.

**Severity:** `issue [blocking]` for missing referenced skills; `suggestion` for missing coordination

### Naming Consistency

**Goal:** Ensure all skill names follow the action+object formula and namespace rules.

**Method:**
1. Check each skill name: it should contain at least one hyphen and follow a verb+noun or verb+adjective+noun pattern (e.g., `upsert-skill`, `create-commit`, `review-pr`).
2. If the library contains 10 or more skills, check whether related skills share a consistent namespace prefix (e.g., `review-pr`, `review-plan`, `review-skill` share `review-`). Flag related skills that lack a shared prefix.
3. Flag names that are single words, noun-only, or do not include an action verb.

**Severity:** `nitpick`

---

## Report Format

Structure the final report in this order:

### 1. Structural Findings

One sub-section per skill with at least one reported finding. Format:

```
## Structural Findings

### <skill-name>

<finding formatted with format-review-comments>
<finding formatted with format-review-comments>
```

Omit skills that pass all checks cleanly. If no skills have findings, write:

```
## Structural Findings

All skills passed structural validation.
```

### 2. Cross-Skill Findings

One sub-section per check category that has findings. Format:

```
## Cross-Skill Findings

### Trigger Conflicts

<finding formatted with format-review-comments>

### Workflow Coverage

<finding formatted with format-review-comments>
```

Omit categories with no findings. If no cross-skill issues are found, write:

```
## Cross-Skill Findings

No cross-skill issues detected.
```

### 3. Coverage Map

Always render the coverage map table, even if all stages are covered:

```
## Coverage Map

| Stage     | Skills          | Status     |
|-----------|-----------------|------------|
| plan      | upsert-plan    | covered    |
| implement | upsert-skill   | covered    |
| commit    | create-commit   | covered    |
| review    | review-pr       | covered    |
| release   | —               | **gap**    |
```

### 4. Overall Summary

```
## Summary

- Skills audited: N
- Auto-fixes applied: N (committed separately)
- Structural findings: N (X bug, Y suggestion, Z nitpick)
- Cross-skill findings: N (X bug, Y suggestion, Z nitpick)
```
