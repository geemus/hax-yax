---
name: create-plan
description: >
  Generates a detailed, structured work plan from a work description, then
  creates or updates a GitHub issue containing the plan. Use when the user wants
  to plan a feature, bug fix, project, or any body of work — also triggered by
  "make a plan", "scope out this work", "break down this task", "help me plan",
  or "create a GitHub issue for this". In update mode (triggered by an issue
  number or URL), edits the issue in place and posts a change-summary comment.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.5"
---

# Plan

Turns a work description into a structured, executable plan and writes it to a GitHub issue. Asks clarifying questions when needed and presents the plan for review before writing. Supports create mode (new issue), update mode (edits an existing issue in place), and breakdown mode (produces a lightweight parent coordination issue with shell sub-issues when complexity signals are present).

## Instructions

### 1. Gather inputs

Collect the following (ask only for what is missing):

- **Work description** — what needs to be done (required; may be provided inline as skill args). If the description is too vague to decompose into concrete tasks without guessing, ask 1–2 targeted clarifying questions before continuing.
- **Existing issue target** — if the inline args contain a bare integer (e.g. `35`), a `#N` reference (e.g. `#35`), or a full GitHub issue URL (e.g. `https://github.com/owner/repo/issues/35`), extract the issue number (and repo from the URL if present). This activates **update mode**; otherwise the skill runs in **create mode**.
- **GitHub repository** — infer `owner/repo` by running `git remote get-url origin` and parsing the result. Parse both HTTPS (`https://github.com/owner/repo.git`) and SSH (`git@github.com:owner/repo.git`) remote formats — for SSH remotes, split on `:` then strip `.git`. If the command exits non-zero, produces empty output, or the result cannot be parsed into `owner/repo`, ask the user for the repository explicitly. Run all git commands without `-C`; the working directory is already the repo root. *(In update mode with a full URL, prefer the repo extracted from the URL.)*
- **Sub-issues** — if the work description clearly involves multiple distinct phases or bodies of work, ask now whether the user wants each phase tracked as a separate sub-issue linked to a parent.

### 1b. Assess complexity

Immediately after gathering inputs, evaluate the work description for the following **complexity signals**:

- Spans multiple systems or teams
- Three or more phases with unresolved inter-phase dependencies
- Significant unknowns that would force guesswork at the task level
- Expected task count greater than 15
- Distinct areas that benefit from separate owners or timing

When **two or more signals are present**, switch to **breakdown mode** instead of proceeding with full planning.

In breakdown mode:
1. Identify the natural planning areas (typically 2–6 named areas such as "Authentication", "Data migration", "API layer").
2. Present the proposed area breakdown to the user and ask for confirmation before continuing. Allow the user to rename, merge, split, or reorder areas.
3. Once confirmed, proceed with the lightweight survey and breakdown templates below.

Skip this step for update mode — complexity was already assessed when the issue was first created.

### 2. Survey the context

Before planning, ground the plan in reality.

**Standard mode:** For a clearly simple task (a single isolated change with no cross-cutting concerns), a lightweight survey — checking only the immediately relevant directory and any context files — is sufficient; skip the open issues/PRs search. For multi-phase or cross-cutting work, run the full survey below.

**Breakdown mode:** Run a lightweight survey only. The goal is to identify and name the planning areas — not to research each area in depth. Check the top-level directory structure, any context files (`CLAUDE.md`, `README.md`, `AGENTS.md`), and search for key terms from the work description. Deep per-area research is deferred to when each sub-plan is created.

- **Existing issue content** *(update mode only)*: Fetch the current title and body of the target issue to use as planning context and to compute a change summary later.
- **Codebase scan**: Use `Glob` to map the directory structure around the relevant area. Use `Grep` to search for key terms from the work description. Check for context files (all optional): `CLAUDE.md`, `README.md`, `AGENTS.md`. Read any that exist for conventions and constraints; if absent, proceed without them.
- **Open issues / PRs** *(skip for simple tasks and breakdown mode)*: Search for open issues and pull requests in the repo matching `<keyword>`, where the keyword is a specific noun or action from the work description that would appear in issue titles. If no issues or PRs exist yet, note the absence and continue.
- **Constraints**: Note tech stack, dependencies, CI requirements, and anything that restricts the approach.
- Record what already exists and what must be built from scratch — this feeds the Background section.

### 3. Consider approaches

**Standard mode:** For non-trivial design decisions, briefly evaluate 2–3 alternative approaches. For each, note the key tradeoff. Select the best approach and document the rationale. This becomes the Approach section. Skip this step only when the work involves fewer than 3 tasks and there is clearly no meaningful choice between approaches.

**Breakdown mode:** Evaluate only top-level structural choices — for example, whether to tackle areas sequentially or in parallel, or which area must be completed first to unblock others. Skip per-area design analysis; that belongs in each sub-plan.

### 4. Generate the plan

**Standard mode:** Produce a plan with these sections:

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

---

**Breakdown mode:** Produce a **parent coordination issue** using the template below, plus one **sub-issue shell** per confirmed planning area.

**Parent coordination issue template:**

```markdown
## Objective
<one or two sentences: the overall goal and why it matters>

## Background
<what exists today, why this work is needed, any top-level constraints>

## Approach
<top-level structural choice: sequencing, parallelism, key dependencies between areas>

## Planning Areas
- [ ] Plan: <Area 1 name> — #<issue> _(no dependencies)_
- [ ] Plan: <Area 2 name> — #<issue> _(depends on: Area 1)_
- [ ] Plan: <Area 3 name> — #<issue> _(parallel with: Area 2)_

## Open Questions
<top-level unknowns; omit section if none>
```

**Sub-issue shell template:**

Title pattern: `<Parent title>: <Area name>` (under 72 characters total).

```markdown
## Scope
**In:** <what this area covers>
**Out:** <what is explicitly excluded>

## Context
<decisions already made that affect this area; outputs expected from prior areas; constraints inherited from the parent>

## Planning Todo
- [ ] Run `/create-plan #N` when ready to plan this area

## Open Questions
<unknowns specific to this area; omit section if none>
```

Each shell is an intentional stub. Do not add a full task list — that is the job of the future sub-plan.

### 5. Self-review before presenting

Evaluate the draft plan against these checks. Fix any issues found silently before presenting to the user.

- Does every task have a clear, actionable description? (If not, rewrite vague tasks.)
- Are dependencies complete and correct?
- Does the acceptance criteria cover the full objective?
- Are any assumptions likely to be wrong?
- Is the plan appropriately scoped — neither too coarse nor too granular?
- Is the Approach section present and does it name the chosen approach, the reason it was selected, and its key tradeoff?
- **Is this plan specifying tasks that depend on unknowns a sub-plan should resolve first?** If yes, consider whether breakdown mode would produce a more accurate result, and flag this for the user if still in standard mode.

### 5.5 Simplify the plan

After self-review, challenge the scope. Ask: is there a simpler way to achieve the same objective? Make changes silently before presenting.

- Remove any task that is not required by the acceptance criteria. If it is "nice to have", add it to Open Questions as a possible extension instead.
- Collapse phases that have no meaningful dependency boundary between them.
- If a simpler approach would satisfy all acceptance criteria with fewer tasks, switch to it and update the Approach section accordingly.
- Flag tasks that could be explicitly deferred post-launch; move them to a "Future work" note at the bottom rather than deleting them.

Do not remove tasks that reflexion identified as missing or necessary.

### 5.8 Refine prose

Apply the `refine-prose` skill to the full plan draft. Run it silently and carry the refined text forward — do not present the pre-refinement draft.

### 6. Present the plan

State the active mode explicitly: **standard mode**, **breakdown mode**, or **update mode**. In breakdown mode, show the parent coordination issue and all sub-issue shells together before creating anything, so the user can review the full structure at once. Present the finished plan (or plans) to the user and ask for confirmation before proceeding.

### 7. Create or update the GitHub issue

Derive the issue title from the Objective: use a concise phrase (under 72 characters) that captures the core action and subject (e.g. "Add rate limiting to the public API").

**Create mode:** Create a new GitHub issue with the derived title and plan body. To avoid shell quoting issues, write the body to a uniquely named temp file first (e.g. `mktemp`) and pass it via a file argument or read it in, then clean up the temp file.

**Update mode:**
1. Update the issue title and body with the new plan. Write the body to a temp file first to avoid shell quoting issues.
2. Compose a change-summary comment that lists each section as **added**, **revised**, **preserved**, or **removed**, with a "Key change" sentence per section. Post it as a comment on the issue.

**Breakdown mode:**
1. Create the parent coordination issue first (body will have placeholder `#TBD` links initially).
2. Create each sub-issue shell in order, using the confirmed area sequence.
3. Edit the parent issue body to replace each `#TBD` placeholder with the actual issue number created in step 2.

**Sub-issues (standard mode)**: If sub-issue tracking was agreed in step 1, create a child issue for each phase using the same method above. Link each child to the parent by adding a line to the parent issue body: `- Sub-issue: #<number> — <phase name>`.

### 8. Report back

**Standard mode:** Share the issue URL, state the number of tasks and phases, and call out any open questions or high-risk assumptions that should be resolved early.

**Breakdown mode:** Share the parent issue URL and the URL of each shell sub-issue. State the number of planning areas, describe the sequencing (which areas are parallel and which are blocked), and call out top-level open questions that affect multiple areas.

**Update mode:** Confirm that the change-summary comment was posted and share the issue URL.

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

**Breakdown mode example:**

Work description: "Migrate the monolith to microservices, including auth, billing, and the data pipeline."

Complexity signals detected: spans multiple systems (3+); 3+ phases with inter-phase dependencies; expected task count well above 15; areas benefit from separate owners.

Claude proposes four planning areas: Auth Service, Billing Service, Data Pipeline, Shared Infrastructure. User confirms. Claude creates:
- Parent issue: "Migrate monolith to microservices" — lists Planning Areas checklist with `#TBD` placeholders, then back-fills once shells are created.
- Shell #42: "Migrate monolith to microservices: Shared Infrastructure" — scope, context (no dependencies), planning todo.
- Shell #43: "Migrate monolith to microservices: Auth Service" — scope, context (depends on Shared Infrastructure #42), planning todo.
- Shell #44: "Migrate monolith to microservices: Billing Service" — scope, context (depends on Shared Infrastructure #42, parallel with Auth Service #43), planning todo.
- Shell #45: "Migrate monolith to microservices: Data Pipeline" — scope, context (depends on Auth #43 and Billing #44), planning todo.

Parent Planning Areas section (after back-fill):
```
- [ ] Plan: Shared Infrastructure — #42 _(no dependencies)_
- [ ] Plan: Auth Service — #43 _(depends on: Shared Infrastructure #42)_
- [ ] Plan: Billing Service — #44 _(depends on: Shared Infrastructure #42, parallel with: Auth Service #43)_
- [ ] Plan: Data Pipeline — #45 _(depends on: Auth Service #43, Billing Service #44)_
```

## Progressive Disclosure

| Level | Content | When to load | Target size |
|-------|---------|--------------|-------------|
| 1 | `name` + `description` | Always (startup) | ~100 tokens |
| 2 | `SKILL.md` body (this file) | When skill is triggered | < 5,000 tokens |
| 3 | None currently defined | — | — |

**Standard mode — generated issue body structure:**
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

**Breakdown mode — generated structure:**
```markdown
<!-- Parent coordination issue -->
## Objective
...

## Background
...

## Approach
...

## Planning Areas
- [ ] Plan: Area 1 — #42 _(no dependencies)_
- [ ] Plan: Area 2 — #43 _(depends on: Area 1 #42)_

## Open Questions
...

<!-- Sub-issue shell (one per area) -->
## Scope
**In:** ...
**Out:** ...

## Context
...

## Planning Todo
- [ ] Run `/create-plan #N` when ready to plan this area

## Open Questions
...
```
