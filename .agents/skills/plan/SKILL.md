---
name: plan
description: >
  Generates a detailed, structured work plan from a description of work to be
  done, then creates or updates a GitHub issue containing the plan. Interactive:
  asks clarifying questions when needed and presents the plan for review. Invoke
  when the user wants to plan out a feature, bug fix, project, or any body of
  work and capture it as a GitHub issue for tracking and collaboration. Also
  supports update mode: when invoked with an existing issue number or URL,
  updates that issue in place and posts a change-summary comment.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.1"
---

# Plan

Turns a description of work into a structured, executable plan written to a GitHub issue. Supports both create mode (new issue) and update mode (edit an existing issue in place).

## Instructions

### 1. Gather inputs

Collect the following (ask only for what is missing):

- **Work description** — what needs to be done (required; may be provided inline as skill args). If the description is too vague to decompose into concrete tasks without guessing, ask 1–2 targeted clarifying questions before continuing.
- **Existing issue target** — if the inline args contain a bare integer (e.g. `35`), a `#N` reference (e.g. `#35`), or a full GitHub issue URL (e.g. `https://github.com/owner/repo/issues/35`), extract the issue number (and repo from the URL if present). This activates **update mode**; otherwise the skill runs in **create mode**.
- **GitHub repository** — infer `owner/repo` by running `git remote get-url origin` and parsing the result. Parse both HTTPS (`https://github.com/owner/repo.git`) and SSH (`git@github.com:owner/repo.git`) remote formats — for SSH remotes, split on `:` then strip `.git`. If the command exits non-zero, produces empty output, or the result cannot be parsed into `owner/repo`, ask the user for the repository explicitly. Run all git commands without `-C`; the working directory is already the repo root. *(In update mode with a full URL, prefer the repo extracted from the URL.)*
- **Sub-issues** — if the work description clearly involves multiple distinct phases or bodies of work, ask now whether the user wants each phase tracked as a separate sub-issue linked to a parent.

### 2. Survey the context

Before planning, ground the plan in reality. For a clearly simple task (a single isolated change with no cross-cutting concerns), a lightweight survey—checking only the immediately relevant directory and any context files—is sufficient; skip the open issues/PRs search. For multi-phase or cross-cutting work, run the full survey below.

- **Existing issue content** *(update mode only)*: Fetch the current title and body of the target issue to use as planning context and to compute a change summary later.
- **Codebase scan**: Use `Glob` to map the directory structure around the relevant area. Use `Grep` to search for key terms from the work description. Check for context files (all optional): `CLAUDE.md`, `README.md`, `AGENTS.md`. Read any that exist for conventions and constraints; if absent, proceed without them.
- **Open issues / PRs** *(skip for simple tasks)*: Search for open issues and pull requests in the repo matching `<keyword>`, where the keyword is a specific noun or action from the work description that would appear in issue titles. If no issues or PRs exist yet, note the absence and continue.
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

**Update mode**: preserve the checked/unchecked state of any task checkboxes that appear in the existing issue. Do not uncheck tasks that were already checked.

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

State whether you are in **create mode** (a new issue will be created) or **update mode** (issue #N will be updated in place). Present the finished plan to the user and ask for confirmation before proceeding.

### 7. Create or update the GitHub issue

Derive the issue title from the Objective: use a concise phrase (under 72 characters) that captures the core action and subject (e.g. "Add rate limiting to the public API").

**Create mode:** Create a new GitHub issue with the derived title and plan body. To avoid shell quoting issues, write the body to a uniquely named temp file first (e.g. `mktemp`) and pass it via a file argument or read it in, then clean up the temp file.

**Update mode:**
1. Update the issue title and body with the new plan. Write the body to a temp file first to avoid shell quoting issues.
2. Compose a change-summary comment that lists each section as **added**, **revised**, **preserved**, or **removed**, with a "Key change" sentence per section. Post it as a comment on the issue.

**Sub-issues**: If sub-issue tracking was agreed in step 1, create a child issue for each phase using the same method above. Link each child to the parent by adding a line to the parent issue body: `- Sub-issue: #<number> — <phase name>`.

### 8. Report back

Share the issue URL, state the number of tasks and phases, and call out any open questions or high-risk assumptions that should be resolved early. In update mode, confirm that the change-summary comment was posted.

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

**Invocation (by issue number — update mode):**
```
/plan #35
```
Fetches issue #35, generates an updated plan, edits the issue in place, and posts a change-summary comment.

**Invocation (by issue URL — update mode):**
```
/plan https://github.com/geemus/skills/issues/35
```
Same as above; repo is extracted from the URL.

## Progressive Disclosure

| Level | Content | When to load | Target size |
|-------|---------|--------------|-------------|
| 1 | `name` + `description` | Always (startup) | ~100 tokens |
| 2 | `SKILL.md` body (this file) | When skill is triggered | < 5,000 tokens |
| 3 | None currently defined | — | — |

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
