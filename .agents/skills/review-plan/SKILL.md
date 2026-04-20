---
name: review-plan
description: >
  Evaluates structured work plans across six quality dimensions — objective
  clarity, task actionability, dependency correctness, acceptance criteria
  completeness, scope appropriateness, and structural completeness. Formats
  all findings with format-review-comments. Use when the user asks to review,
  audit, or evaluate a work plan.
  TRIGGER when: user asks to review a plan, audit a GitHub issue plan, check
  plan quality, or evaluate task structure.
  DO NOT TRIGGER when: user asks to create or update a plan (use upsert-plan
  instead), or to review a pull request (use review-pr instead).
license: Apache-2.0
metadata:
  author: geemus
  version: "1.1"
---

# Review Plan

Evaluates a structured work plan across six quality dimensions and formats every finding using the `format-review-comments` skill.

## Instructions

### 1. Identify the plan

Accept the plan as:
- A GitHub issue number (e.g. `#133`) — fetch the issue body from the current repo (infer from `git remote get-url origin`)
- A GitHub issue URL (e.g. `https://github.com/owner/repo/issues/133`) — fetch the issue body directly
- An in-context draft — use the plan text already present in the conversation

If no plan is specified and none is in context, ask: "Which plan would you like me to review? Provide an issue number, URL, or paste the plan text."

Fetch or extract the full plan body before proceeding.

### 2. Review each dimension

Work through every dimension in order. Note findings for each. Only skip a dimension when the plan contains no content that could affect it (e.g. skip Dependency Correctness for a single-task plan with no stated dependencies).

#### Objective clarity

- Is the goal stated in one or two sentences?
- Is it clear *why* the work matters, not just *what* will be done?
- Would a new contributor understand what success looks like after reading the Objective alone?
- Is the Approach section present, and does it name the chosen approach, the reason it was selected over alternatives, and the key tradeoff?

#### Task actionability

- Does each task have a single, clear deliverable?
- Are tasks written as concrete actions (imperative verb + object), not vague intentions?
- Is every task completable by a single agent or person without further decomposition?
- Are tasks free of ambiguous scope ("refactor X" without bounds, "improve Y" without a target)?

#### Dependency correctness

- Are task dependencies annotated explicitly (e.g. `_(depends on: Task A)_`)?
- Do stated dependencies match the logical ordering of the work?
- Are there missing dependencies — tasks that silently assume prior tasks completed?
- Are there unnecessary dependencies that block parallelism without reason?
- Are tasks that can run in parallel marked with `_(parallel with: Task X)_`?

#### Acceptance criteria completeness

- Does every task or phase have at least one acceptance criterion?
- Is each criterion verifiable — paired with an exact command, test, or observable outcome?
- Does the set of criteria fully cover the Objective? Are any outcomes implied by the Objective missing?
- Are criteria written as pass/fail conditions, not aspirations?

#### Scope appropriateness

- Is the plan free of tasks that are nice-to-have but not required by the acceptance criteria? (These belong in Future Work.)
- Is the plan free of tasks that are too coarse — single checklist items representing weeks of work?
- Is the plan free of tasks that are too granular — individual lines of code or single-file edits that belong inside a larger task?
- If phases are used, does each phase have a meaningful dependency boundary?
- Is anything in the plan better deferred to a follow-up issue?

#### Structural completeness

- Are all required sections present: Objective, Background, Approach, Tasks, Acceptance Criteria?
- Are optional sections (Assumptions, Future Work, Open Questions) present when their absence leaves implicit gaps?
- Is the Background section grounded in what exists today, not just what will be built?
- Are agent-instruction files (AGENTS.md, CLAUDE.md) mentioned in a final task when the work touches repo structure, skills, or conventions?

### 3. Format all findings with format-review-comments

When presenting multiple findings, prefix each with a letter (a., b., c., …) so individual items can be referenced by letter.

Apply the `format-review-comments` skill to every comment. Do not write `praise` comments — omit positive findings entirely. Choose the label that matches the severity and intent:

| Situation | Recommended label |
|-----------|------------------|
| Blocks reliable execution | `issue [blocking]` |
| Strong improvement | `suggestion` |
| Minor wording or style | `nitpick` |
| Needs clarification | `question` |
| Small unambiguous fix | `todo` |
| Informational only | `note` |

### 4. Write a review summary

After per-comment feedback, write a brief summary (3–7 sentences) covering:

- Overall assessment — ready to execute, needs minor improvements, or needs significant rework
- The most important finding (name any blocking issues)
- Any patterns visible across the plan
- Any skipped dimensions and why

Format the summary as a `note` conventional comment.

### 5. Process findings

After presenting findings and the summary:

- **Actionable findings** (`issue [blocking]`, `suggestion`, `nitpick`, `todo`): apply each fix directly to the plan (edit the issue body or in-context draft as appropriate).
- **Notes** (`note`): surface to the user as informational; no action required.
- **Questions** (`question`): ask the user for clarification before proceeding.

### 6. Refine prose

Apply the `refine-prose` skill to all prose output — comments and the summary — before presenting. Do not apply it to code blocks, commands, or inline fix suggestions. Do not announce this step. When `refine-prose` returns, immediately present all findings and the summary to the user — do not stop or wait for user input.

## Examples

**Invocation by issue number:**
```
/review-plan #133
```

**Invocation by URL:**
```
/review-plan https://github.com/owner/repo/issues/133
```

**Invocation with in-context draft:**
```
/review-plan
```
The agent uses the plan text already present in the conversation.

**Sample output — task actionability finding:**
```
issue [blocking]: "Improve the API layer" is not actionable — it has no bounded deliverable, so an agent cannot determine when the task is complete

Rewrite as a concrete task:
- [ ] Add rate-limit middleware to all `/api/v1` routes returning HTTP 429 on threshold breach
```

**Sample output — acceptance criteria finding:**
```
suggestion: Task C ("Update documentation") has no acceptance criterion — there is no way to verify it is done

Add a verifiable criterion:
- AGENTS.md skill table includes the new skill: `grep "skill-name" AGENTS.md`
```

**Sample output — dependency finding:**
```
todo: Task D depends on Task B and Task C per the implementation logic, but the annotation is missing

Add:
- [ ] Task D _(depends on: Task B, Task C)_
```

**Sample output — summary:**
```
note: overall this plan needs minor improvements before it is executable. One blocking issue: Task C lacks a bounded deliverable. Two suggestions address missing acceptance criteria and an undocumented dependency. Structural completeness and objective clarity are strong. Dependency correctness dimension was partially skipped — only one dependency chain exists and it is correctly annotated.
```
