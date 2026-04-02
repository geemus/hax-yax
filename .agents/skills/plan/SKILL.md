---
name: plan
description: >
  Generates a detailed, structured work plan from a description of work to be
  done, then creates a GitHub issue containing the plan. Invoke when the user
  wants to plan out a feature, bug fix, project, or any body of work and
  capture it as a GitHub issue for tracking and collaboration.
license: Apache-2.0
metadata:
  author: geemus
  version: "2.1"
allowed-tools:
  - Bash
  - Glob
  - Grep
  - Read
  - mcp__github__list_issues
  - mcp__github__list_pull_requests
  - mcp__github__issue_write
---

# Plan

Turns a description of work into a structured, executable plan written to a GitHub issue.

## Instructions

### 1. Gather inputs

Collect the following (ask only for what is missing):

- **Work description** — what needs to be done (required; may be provided inline as skill args)
- **GitHub repository** — infer `owner/repo` by running `git remote get-url origin` and parsing the result; only ask if the remote cannot be determined
- **Issue title** — generate one from the description if not provided
- **Labels** — if not provided, suggest defaults based on the label strategy in step 6; confirm with the user before applying

### 2. Survey the context

Before planning, ground the plan in reality:

- Scan the codebase for existing patterns, conventions, and related code relevant to the work
- Check for open issues or PRs related to the same area (`mcp__github__list_issues`, `mcp__github__list_pull_requests`)
- Note any constraints (tech stack, dependencies, CI requirements, etc.)
- Record what already exists and what must be built from scratch — this becomes the Background section

### 3. Consider approaches

Before committing to a plan, briefly evaluate 2–3 alternative approaches. For each, note the key tradeoff. Select the best approach and document the rationale. This becomes the Approach section in the issue.

### 4. Generate the plan

Produce a plan with these sections:

#### Objective
One or two sentences stating the goal and why it matters.

#### Background
What exists today, what the context survey found, why this work is needed, and any constraints.

#### Approach
The chosen approach and why it was selected over alternatives. One short paragraph; include the key tradeoff.

#### Assumptions
Explicit statements the plan depends on being true (e.g. "The existing auth middleware can be reused", "No breaking API changes are needed"). Flag anything that, if wrong, would require replanning.

#### Tasks
Ordered, actionable tasks as GitHub Markdown checkboxes. For non-trivial work, group into phases. Each task should have a single, clear deliverable.

Annotate dependencies explicitly:

```
- [ ] Task A — no dependencies
- [ ] Task B — no dependencies
- [ ] Task C _(depends on: Task A)_
  - [ ] Sub-task
- [ ] Task D _(depends on: Task B, Task C)_
```

Mark tasks that can run in parallel with a note: _(parallel with: Task X)_.

#### Acceptance Criteria
A bulleted list of pass/fail conditions. Each criterion must be verifiable. Pair each with the exact command or action used to verify it:

```
- All existing tests pass: `npm test`
- Rate limit headers present on API responses: `curl -I /api/... | grep X-RateLimit`
```

#### Open Questions
Unknowns that need resolution before or during the work. Distinct from Assumptions: these are things the plan genuinely cannot answer yet. Omit this section if there are none.

### 5. Self-review before posting

Before creating the issue, evaluate the draft plan:

- Does every task have a clear, actionable description? (If not, rewrite vague tasks.)
- Are dependencies complete and correct?
- Does the acceptance criteria cover the full objective?
- Are any assumptions likely to be wrong?
- Is the plan appropriately scoped — neither too coarse nor too granular?

Fix any issues found, then proceed.

### 6. Create the GitHub issue

Use `mcp__github__issue_write` with:
- `owner` and `repo` from step 1
- `title` from step 1
- `body` as the full Markdown plan from step 4
- `labels` from step 1

**Label strategy** — apply one label from each applicable category:
- *Type*: `feature`, `bug`, `refactor`, `docs`, `test`
- *Priority*: `priority: high`, `priority: medium`, `priority: low`

**Sub-issues**: If the plan has more than ~7 tasks or multiple distinct phases, suggest to the user that each phase (or major task group) be tracked as a separate sub-issue linked to this parent. Offer to create them.

### 7. Report back

Share the issue URL, state the number of tasks and phases, and call out any open questions or high-risk assumptions that should be resolved early.

## Examples

**Invocation (inline description):**
```
/plan Add rate limiting to the public API
```

**Invocation (interactive):**
```
/plan
```
Claude will ask for the work description. Repository is inferred from `git remote get-url origin`.

**Generated issue body structure:**
```markdown
## Objective
...

## Background
...

## Approach
We will use approach X because it minimizes Y; the key tradeoff is Z.

## Assumptions
- ...

## Tasks

### Phase 1 — Research
- [ ] Task A
- [ ] Task B _(parallel with: Task A)_

### Phase 2 — Implementation
- [ ] Task C _(depends on: Task A)_
  - [ ] Sub-task

### Phase 3 — Validation
- [ ] Task D _(depends on: Task C)_

## Acceptance Criteria
- All tests pass: `npm test`
- ...

## Open Questions
- ...
```
