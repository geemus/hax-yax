---
name: implement-plan
description: >
  Reads a structured plan from a GitHub issue and orchestrates the full
  delivery workflow: parses unchecked tasks in dependency order, steps through
  them one at a time, commits via create-commit, reviews via review-pr, and
  opens a PR linked back to the plan issue.
  Use when the user says "implement this plan", "execute plan #N", "work
  through issue #N", "implement the plan in #N", or "carry out the tasks in
  #N".
  TRIGGER when: user wants to execute or implement a plan stored as a GitHub
  issue.
  DO NOT TRIGGER when: user wants to create or update a plan (use upsert-plan
  instead).
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Implement Plan

Fetches a structured plan from a GitHub issue, steps through each unchecked task in dependency order, commits the result, runs a PR review, and opens a PR linked back to the plan issue. Composes `create-commit` and `review-pr` rather than reimplementing their logic.

## Instructions

**Side effects:** implements code changes, creates a git commit, may open a GitHub PR. Task checkboxes on the plan issue are left unchecked throughout, allowing the skill to be re-run or retried.

### 1. Accept input and fetch the plan issue

Accept the plan as:
- A bare integer (e.g. `126`) — a GitHub issue number; infer the repo from `git remote get-url origin`
- A `#N` reference (e.g. `#126`)
- A full GitHub issue URL (e.g. `https://github.com/owner/repo/issues/126`)

If no plan is specified, ask: "Which plan issue should I implement? Provide an issue number, `#N` reference, or full URL."

Fetch the issue body using the GitHub tools available in the environment. If the issue cannot be fetched, report the error and stop.

### 2. Parse and present unchecked tasks

Extract all unchecked checkboxes (`- [ ]`) from the issue body, preserving their order and any phase groupings. Checked tasks (`- [x]`) are skipped — they represent already-completed work.

Respect dependency annotations (e.g. `_(depends on: Task A)_`) to surface tasks in an order that satisfies all declared dependencies. When tasks are marked `_(parallel with: Task X)_`, implement them sequentially but note they can be batched in future runs.

Present the task list to the user before starting:
- State the plan issue number and title
- List all unchecked tasks grouped by phase (if phases exist)
- Call out any tasks blocked by incomplete dependencies
- Ask for confirmation: "Ready to implement these N tasks. Proceed?"

Do not start implementation until the user confirms.

### 3. Implement each task in order

For each unchecked task (in dependency-resolved order):

1. State which task you are beginning (e.g. "Starting: Create `.agents/skills/implement-plan/SKILL.md`").
2. Implement the task using the tools available (Read, Write, Edit, Bash, Glob, Grep, GitHub tools, etc.).
3. Report completion: "Done: `<task description>`."
4. If a task cannot be completed (missing dependency, ambiguity, external blocker), stop and ask the user for guidance before continuing.

**Do not** check off task checkboxes on the plan issue — leave them unchecked so the skill can be retried or re-run without losing progress markers.

After all tasks are implemented, summarize what was done before proceeding.

### 4. Commit via create-commit

Invoke the `create-commit` skill (`/commit`) to stage and commit all changes made during step 3.

Wait for the commit to succeed before continuing. If `create-commit` reports nothing to commit, note this and proceed to step 5.

### 5. Review via review-pr

Invoke the `review-pr` skill on the current branch against the base branch (default: `main`):

```
/review-pr <current-branch>
```

Process the review findings:
- **Blocking findings** (`issue [blocking]`): fix each one immediately, then re-run `create-commit` to commit the fixes before proceeding.
- **Non-blocking findings** (`suggestion`, `nitpick`, `question`, `todo`, `note`): surface them to the user and ask: "These non-blocking findings were identified. Would you like me to address any of them before opening the PR?"

Do not proceed to step 6 until all blocking findings are resolved.

### 6. Open a PR linked to the plan issue

Derive the PR title from the plan issue title: keep it concise (under 70 characters) and imperative (e.g. "Add implement-plan skill to orchestrate plan execution").

Create the PR using the GitHub tools available in the environment, linking back to the plan issue in the body:

```markdown
## Summary
<bullet points summarizing what was implemented>

## Plan
Implements #<plan-issue-number>

## Test plan
<bulleted checklist of how to verify the changes>
```

Report the PR URL to the user.

## Examples

**Invocation with issue number:**
```
/implement-plan 126
```

**Invocation with `#N` reference:**
```
/implement-plan #126
```

**Invocation with URL:**
```
/implement-plan https://github.com/geemus/hax-yax/issues/126
```

**Invocation (interactive):**
```
/implement-plan
```
The agent will ask which plan issue to implement.

**Sample task presentation:**
```
Plan: #126 — Add implement-plan skill to orchestrate plan execution

Phase 1 — Design (2 tasks):
  - [ ] Define task-parsing contract
  - [ ] Confirm checkpointing behavior

Phase 2 — Implementation (5 tasks):
  - [ ] Create .agents/skills/implement-plan/SKILL.md  _(depends on: Phase 1)_
  - [ ] Write Step 1 instructions  _(depends on: Phase 1)_
  - [ ] Write Step 2 instructions  _(depends on: Step 1)_
  - [ ] Write Step 3 instructions  _(depends on: Step 2)_
  - [ ] Write Step 4 instructions  _(depends on: Step 3)_

Phase 3 — Housekeeping (1 task):
  - [ ] Run upsert-agents-md  _(depends on: Phase 2)_

Ready to implement these 8 tasks. Proceed?
```

**Sample blocking-finding fix:**
```
review-pr found 1 blocking issue:
  issue [blocking]: `SKILL.md` frontmatter is missing the required `name` field

Fixing automatically…
[edits file]
Committing fix via create-commit…
[commits]
Proceeding to open PR.
```
