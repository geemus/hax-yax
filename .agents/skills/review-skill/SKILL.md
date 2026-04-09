---
name: review-skill
description: >
  Evaluates whether a skill is effective — covering instruction clarity, trigger
  discoverability, example completeness, composability, and scope coherence.
  Reads the full skill directory including supporting files (scripts, references, assets) before reviewing.
  Formats every finding with format-review-comments and suggests inline fixes
  for unambiguous issues. Use when the user asks to review, audit, or evaluate
  a skill.
  TRIGGER when: user asks to review a skill, audit a skill, evaluate skill
  quality, or check if a skill is well-written.
  DO NOT TRIGGER when: user asks to create, update, rename, or delete a skill
  (use upsert-skill instead), or to review a pull request (use review-pr
  instead).
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Review Skill

Evaluates a skill for effectiveness — not just structural validity — across five quality dimensions, and formats every finding using the `format-review-comments` skill.

## Instructions

### 1. Identify the target skill

Accept the skill as:
- A skill name (e.g. `create-plan`) — resolve to `.agents/skills/<name>/`
- A path (e.g. `.agents/skills/create-plan/`) — use directly

Confirm the skill directory exists before proceeding. If it does not exist, stop and tell the user: "Skill directory `<path>` not found. Provide a valid skill name or path."

### 2. Read the skill in full

Read `SKILL.md` first. Then read every Level 3 file present:
- `scripts/` — all executable scripts
- `references/` — all reference documents
- `assets/` — all templates and examples

Do not begin reviewing until all files have been read. A finding that misses information in Level 3 files is an incomplete finding.

### 3. Review each dimension

Work through every dimension in order. For each, apply the evaluation questions and note findings. Only example completeness is skippable — skip it when the skill is a single-step utility with self-evident output. All other four dimensions are mandatory. For each finding, note whether an inline fix is clear and unambiguous.

#### Instruction clarity

- Are the steps numbered and ordered so an agent can follow them without inferring intent?
- Is every term defined on first use, or is prior knowledge assumed without justification?
- Are conditional branches (if/else paths) explicit, or does the agent have to guess?
- Do instructions tell the agent *what to do* rather than *what to think about*?
- Are external dependencies (tools, scripts, environment variables) named explicitly?

#### Trigger discoverability

- Does the `description` open with the action verb and clearly state what the skill does?
- Are the natural-language trigger phrases an agent or user would actually say included?
- Are synonyms and alternate phrasings covered (e.g. both "commit" and "save changes")?
- If the skill overlaps with another skill, does the description include `TRIGGER when` and `DO NOT TRIGGER when` lines that unambiguously separate them?
- Is the description under 1024 characters?

#### Example completeness

- Is there at least one invocation example showing the most common usage?
- Do examples cover non-obvious or alternate invocation forms (e.g. by path vs. by name)?
- Does at least one example show formatted output so the reviewer knows what to expect?
- Do examples cover edge cases or failure modes when they are likely in practice?

#### Composability

- Does the skill invoke other skills where appropriate (e.g. `format-review-comments`, `refine-prose`), and are those invocations explicit in the instructions?
- Are side effects (files written, commands run, external services called) documented so callers can reason about sequencing?
- Does the skill produce output in a form that downstream skills or steps can consume?
- Is the skill scoped narrowly enough to be reused, or does it bundle concerns that belong in separate skills?

#### Scope coherence

- Does the skill name match what the skill actually does (action + object, outcome-focused)?
- Is there a single clear purpose, or does the skill attempt to do multiple unrelated things?
- Is anything in the instructions clearly out of scope and better delegated to another skill?
- Are destructive operations guarded by an explicit confirmation step?

### 4. Format all findings with format-review-comments

Apply the `format-review-comments` skill to every comment. Choose the label that matches the severity and intent:

| Situation | Recommended label |
|-----------|------------------|
| Blocks reliable agent use | `issue [blocking]` |
| Strong improvement | `suggestion` |
| Minor style or wording | `nitpick` |
| Needs clarification | `question` |
| Small unambiguous fix | `todo` |
| Informational only | `note` |

Do not write `praise` comments. Omit positive findings entirely — they add tokens without actionable content.

### 5. Suggest inline fixes for unambiguous findings

For any finding where the correct fix is clear — rewritten instruction text, corrected frontmatter, improved example — include the suggested replacement inline with the comment so the reviewer can apply it directly. Do not suggest fixes for findings that require a judgment call.

### 6. Write a review summary

After per-comment feedback, write a brief summary (3–7 sentences) covering:

- Overall assessment — effective as-is, needs minor improvements, or needs significant rework
- The most important finding (name any blocking issues)
- Any patterns visible across the skill
- Any skipped dimensions and why

Format the summary as a `note` conventional comment.

### 7. Refine prose

Apply the `refine-prose` skill to all output before presenting it. Do not announce this step to the user.

## Examples

**Invocation by skill name:**
```
/review-skill create-plan
```

**Invocation by path:**
```
/review-skill .agents/skills/create-plan/
```

**Sample output — skill not found:**
```
Skill directory `.agents/skills/nonexistent/` not found. Provide a valid skill name or path.
```

**Sample output — instruction clarity finding:**
```
issue [blocking]: step 3 says "process the results appropriately" without defining what appropriate means — an agent will make inconsistent choices

Replace with explicit instructions:
> 3. For each result, extract the `title` and `url` fields and write them as a Markdown list item: `- [title](url)`
```

**Sample output — trigger discoverability finding:**
```
suggestion: the description does not include `DO NOT TRIGGER when` lines to disambiguate from `upsert-skill`, so an agent may invoke the wrong skill when the user says "check my skill"

Add after the existing description:
> TRIGGER when: user asks to review, audit, or evaluate a skill.
> DO NOT TRIGGER when: user asks to create, update, or delete a skill (use upsert-skill instead).
```

**Sample output — inline fix for a nitpick:**
```
nitpick: description opens with "This skill evaluates…" — passive opener wastes the first tokens agents read

Change to:
> Evaluates whether a skill is effective…
```

**Sample output — summary:**
```
note: overall this skill needs minor improvements before it is reliably discoverable. One blocking issue: step 4 leaves agent behavior undefined for empty inputs. Two suggestions address trigger disambiguation and example coverage. Instruction clarity and scope coherence are strong. Example completeness was partially skipped — the skill is a single-step utility where output format is self-evident.
```
