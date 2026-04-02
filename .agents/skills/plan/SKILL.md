---
name: plan
description: >
  Generates a detailed, structured work plan from a description of work to be
  done, then creates a GitHub issue containing the plan. Interactive: asks
  clarifying questions when needed and presents the plan for review. Invoke when the user wants to plan out a
  feature, bug fix, project, or any body of work and capture it as a GitHub
  issue for tracking and collaboration.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Plan

Turns a description of work into a structured, executable plan written to a GitHub issue.

## Instructions

### 1. Gather inputs

Collect the following (ask only for what is missing):

- **Work description** — what needs to be done (required; may be provided inline as skill args). If the description is too vague to decompose into concrete tasks without guessing, ask 1–2 targeted clarifying questions before continuing.
- **GitHub repository** — infer `owner/repo` by running `git remote get-url origin` and parsing the result. Parse both HTTPS (`https://github.com/owner/repo.git`) and SSH (`git@github.com:owner/repo.git`) remote formats — for SSH remotes, split on `:` then strip `.git`. If the command exits non-zero, produces empty output, or the result cannot be parsed into `owner/repo`, ask the user for the repository explicitly.
- **Sub-issues** — if the work description clearly involves multiple distinct phases or bodies of work, ask now whether the user wants each phase tracked as a separate sub-issue linked to a parent.

Check `gh` availability once here and carry the result forward — do not re-check in later steps:
```
gh auth status 2>/dev/null && echo "gh: available" || echo "gh: unavailable"
```

### 2. Survey the context

Before planning, ground the plan in reality:

- **Codebase scan**: Use `Glob` to map the directory structure around the relevant area. Use `Grep` to search for key terms from the work description. Read `AGENTS.md`, `CLAUDE.md`, and `README.md` if present for conventions and constraints. If none of these files exist, note the absence and continue.
- **Open issues / PRs**: Check for related work using whichever path was determined in step 1. Choose a specific keyword from the work description (a noun or action that would appear in issue titles) and use it consistently:
  - `gh` available: `gh issue list --repo <owner/repo> --state open --search "<keyword>"` and `gh pr list --repo <owner/repo> --state open --search "<keyword>"`
  - `gh` unavailable: `mcp__github__list_issues` / `mcp__github__list_pull_requests`
  - If no issues or PRs exist yet, note the absence and continue.
- **Constraints**: Note tech stack, dependencies, CI requirements, and anything that restricts the approach.
- Record what already exists and what must be built from scratch — this feeds the Background section.

### 3. Consider approaches

For non-trivial design decisions, briefly evaluate 2–3 alternative approaches. For each, note the key tradeoff. Select the best approach and document the rationale. This becomes the Approach section. Skip this step only when the work involves fewer than 3 tasks and there is clearly no meaningful choice between approaches.

### 4. Generate the plan

Produce a plan with these sections:

#### Objective
One or two sentences stating the goal and why it matters.

#### Background
What exists today, what the context survey found, why this work is needed, and any constraints.

#### Approach
The chosen approach and why it was selected over alternatives. One short paragraph; include the key tradeoff.

#### Assumptions
Explicit statements the plan depends on being true (e.g. "The existing auth middleware can be reused", "No breaking API changes are needed"). Flag anything that, if wrong, would require replanning. Omit this section if there are none.

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

### 5. Self-review before presenting

Evaluate the draft plan against these checks. Fix any issues found silently before presenting to the user.

- Does every task have a clear, actionable description? (If not, rewrite vague tasks.)
- Are dependencies complete and correct?
- Does the acceptance criteria cover the full objective?
- Are any assumptions likely to be wrong?
- Is the plan appropriately scoped — neither too coarse nor too granular?
- Is the Approach section present and does it name the chosen approach, the reason it was selected, and its key tradeoff?

### 6. Present the plan

Present the finished plan to the user and ask for confirmation before creating the issue.

### 7. Create the GitHub issue

Derive the issue title from the Objective: use a concise phrase (under 72 characters) that captures the core action and subject (e.g. "Add rate limiting to the public API").

Use the path determined in step 1:

- **`gh` available:** Write the plan body to a uniquely named temp file using the `Write` tool (avoids shell quoting issues and concurrent collisions), then create the issue and clean up the temp file:
  ```
  EPOCH=$(date +%s)
  gh issue create --repo <owner/repo> --title "<derived title>" --body-file /tmp/plan-body-${EPOCH}.md
  rm /tmp/plan-body-${EPOCH}.md
  ```

- **`gh` unavailable:** Use `mcp__github__create_issue` with `owner`, `repo`, `title`, and `body`.

**Sub-issues**: If sub-issue tracking was agreed in step 1, create a child issue for each phase using the same method above. Link each child to the parent by adding a line to the parent issue body: `- Sub-issue: #<number> — <phase name>`. If `mcp__github__sub_issue_write` is available, use it to create the formal parent–child relationship.

### 8. Report back

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
```
