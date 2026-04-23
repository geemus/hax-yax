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
  version: "1.1"
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

### 3. Implement each task in order

For each unchecked task (in dependency-resolved order):

1. State which task you are beginning (e.g. "Starting: Create `.agents/skills/implement-plan/SKILL.md`").
2. Implement the task using the tools available (Read, Write, Edit, Bash, Glob, Grep, GitHub tools, etc.).
3. Report completion: "Done: `<task description>`."
4. If a task cannot be completed (missing dependency, ambiguity, external blocker), stop and ask the user for guidance before continuing.

**Defer all lint, test, and typecheck runs** until the dedicated final-validation step (step 6). In this step, do not run lint, tests, or typecheck after individual tasks — intermediate states are not the validation target, and per-task runs accumulate redundant work across the loop. All such commands are deferred to step 6, where they execute exactly once against the final post-review state. The only exception is running a command to confirm a task's concrete output (e.g. a generator script the task explicitly requires); even then, do not invoke repo-wide lint or test suites.

**Do not** check off task checkboxes on the plan issue — leave them unchecked so the skill can be retried or re-run without losing progress markers.

After all tasks are implemented, write a brief summary of what was done, then immediately proceed to step 4 — do not stop or wait for user input.

### 4. Commit via create-commit

Immediately after all tasks are complete, invoke the `create-commit` skill (`/commit`) — do not pause, summarize and stop, or ask for confirmation. Proceed directly.

Wait for the commit to succeed, then proceed immediately to step 5. If `create-commit` reports nothing to commit, note this and proceed immediately to step 5 — do not stop.

### 5. Review via review-pr

Immediately after the commit in step 4 completes, invoke the `review-pr` skill on the current branch against the base branch (default: `main`) — do not pause or ask for confirmation:

```
/review-pr <current-branch>
```

`review-pr` will automatically fix all actionable findings and commit them. **Do not run lint, tests, or typecheck during `review-pr` fix iteration** — lint and tests are deferred to step 6, where they run exactly once on the final post-review state. Validating against each intermediate fix-and-recommit cycle is redundant, since only the final state ships. When `review-pr` completes, proceed immediately to step 6 — do not stop or wait for user input.

### 6. Run lint and tests on final state

Immediately after `review-pr` completes, run the repository's lint, typecheck, and test commands exactly once against the final post-review state — do not pause or ask for confirmation. This is the single validation gate; per-task and per-review runs were deferred here specifically to avoid redundant validation of intermediate states.

**Command discovery** (in priority order — use the first source that defines each command):
1. `AGENTS.md` / `CLAUDE.md` — look for an explicit "Testing", "Linting", or "Validation" section naming the commands to run
2. `package.json` `scripts` — prefer `lint`, `typecheck`, `test` (in that order); fall back to `check` or `ci` if they exist
3. `Makefile` — prefer `make lint`, `make typecheck`, `make test`
4. Language defaults — `ruff check . && pytest` (Python), `cargo clippy && cargo test` (Rust), `go vet ./... && go test ./...` (Go), etc.

If none of these are present and no convention is documented, skip this step and note it in the PR body.

**Execution order:** lint → typecheck → tests. Stop at the first failure; a failing lint run typically masks real type or test errors.

**Fix-and-recommit loop** (capped at **2 retries**):
1. If a command fails, inspect the output and fix the root cause — do not suppress warnings or disable rules to make them pass.
2. Invoke `create-commit` to commit the fix.
3. Re-run the failed command (and any later commands in the order above).
4. Repeat up to 2 times total.

**Escalation:** if failures persist after 2 fix attempts, or if a failure is clearly outside the scope of this plan (e.g. a pre-existing broken test unrelated to the changed files), stop and report to the user:
- which command failed
- the relevant error excerpt
- what was tried
- whether the agent believes the failure is in-scope or pre-existing

When all commands pass (or the step is skipped per the rules above), proceed immediately to step 7 — do not stop or wait for user input.

### 7. Open a PR linked to the plan issue

Immediately after step 6 completes, derive the PR title from the plan issue title and open the PR — do not pause or ask for confirmation.

Keep the title concise (under 70 characters) and imperative (e.g. "Add implement-plan skill to orchestrate plan execution").

Create the PR using the GitHub tools available in the environment, linking back to the plan issue in the body:

```markdown
## Summary
<bullet points summarizing what was implemented>

## Plan
Closes #<plan-issue-number>

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
Proceeding to step 6 (lint and tests on final state).
```

**Sample lint/test failure recovery:**
```
Step 6 — running lint and tests on final post-review state.

Discovered commands from package.json scripts: `npm run lint`, `npm test`.

Attempt 1:
  $ npm run lint
  ✗ src/plan.ts:42  no-unused-vars: 'tmp' is defined but never used

Fix attempt 1/2: remove unused `tmp` binding in src/plan.ts:42.
Committing fix via create-commit…
[commits]

Attempt 2:
  $ npm run lint
  ✓ passed
  $ npm test
  ✓ 47 passed

All validation commands pass. Proceeding to step 7 (open PR).
```

**Sample full flow (happy path):**
```
Step 3 — implementing 4 tasks, deferring all lint/test runs to step 6.
[task 1 done] [task 2 done] [task 3 done] [task 4 done]

Step 4 — /commit
Step 5 — /review-pr claude/implement-plan-126 (no blocking findings)
Step 6 — lint + typecheck + tests on final state (all pass)
Step 7 — opening PR
```
