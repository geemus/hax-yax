---
name: work-plan
description: >
  Generates a detailed, structured work plan from a description of work to be
  done, then creates a GitHub issue containing the plan. Invoke when the user
  wants to plan out a feature, bug fix, project, or any body of work and
  capture it as a GitHub issue for tracking and collaboration.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.1"
---

# Work Plan

Turns a description of work into a structured plan written to a GitHub issue.

## Instructions

### 1. Gather inputs

Collect the following from the user (ask only for what is missing):

- **Work description** — what needs to be done (required; may be provided inline as skill args)
- **GitHub repository** — infer `owner/repo` from the current git context by running `git remote get-url origin` and parsing the result; only ask the user if the remote cannot be determined
- **Issue title** — short summary; generate one from the description if not provided
- **Labels** — optional comma-separated list of labels to apply to the issue

### 2. Generate the plan

Think through the work carefully and produce a plan with these sections:

#### Objective
One or two sentences stating the goal and why it matters.

#### Background
Relevant context: what exists today, why the work is needed, any constraints.

#### Tasks
A numbered, ordered list of concrete tasks. For non-trivial work, group tasks into phases (e.g. *Phase 1 — Research*, *Phase 2 — Implementation*, *Phase 3 — Validation*). Each task should be actionable and independently completable. Use sub-tasks (indented checkboxes) for steps within a task.

Format tasks as GitHub Markdown task lists:
```
- [ ] Task description
  - [ ] Sub-task
```

#### Acceptance Criteria
A bulleted list of conditions that must be true for the work to be considered complete. Write each criterion so it can be evaluated as pass/fail.

#### Open Questions
Any unknowns, risks, or decisions that need resolution before or during the work. Leave this section empty (or omit it) if there are none.

### 3. Create the GitHub issue

Use the GitHub MCP tool `mcp__github__create_issue` with:
- `owner` and `repo` from the repository input
- `title` from step 1
- `body` as the full Markdown plan from step 2
- `labels` if provided

### 4. Report back

Share the created issue URL with the user and briefly summarize the plan structure (number of tasks, phases if any).

## Examples

**Invocation (inline description):**
```
/work-plan Add rate limiting to the public API
```

**Invocation (interactive):**
```
/work-plan
```
Claude will ask for the work description. The repository is inferred automatically from `git remote get-url origin`.

**Generated issue body structure:**
```markdown
## Objective
...

## Background
...

## Tasks

### Phase 1 — Research
- [ ] ...

### Phase 2 — Implementation
- [ ] ...

## Acceptance Criteria
- ...

## Open Questions
- ...
```
