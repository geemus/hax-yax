---
name: review-pr
description: >
  Guides systematic pull request review across logic, security, performance,
  test coverage, and documentation. Uses the format-review-comments skill to
  format all feedback. Invoke when the user asks to review a pull request,
  audit a diff, or give structured feedback on proposed code changes.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.0"
---

# Review PR

Conducts a structured, multi-dimensional review of a pull request and formats every comment using the `format-review-comments` skill.

## Instructions

### 1. Identify the PR

Accept the PR as:
- A GitHub PR URL (e.g. `https://github.com/owner/repo/pull/42`)
- A bare PR number (e.g. `42`) — infer the repo from `git remote get-url origin`
- A local diff or branch name — use `git diff <base>..<head>` to obtain the diff

Fetch the PR diff and description before proceeding.

### 2. Read the PR description

Understand the stated intent: what problem does the PR solve, what approach was taken, and what is explicitly out of scope? Use this as the lens for all review dimensions below.

### 3. Review each dimension

Work through every dimension in order. For each, read the relevant changed files and note findings. You do not need to comment on every dimension — skip a dimension only when it is genuinely not applicable to the changes (e.g. no security surface touched → skip security).

#### Logic and correctness

- Does the code do what the PR description claims?
- Are there off-by-one errors, incorrect conditions, or missed edge cases?
- Are error paths handled correctly?
- Are concurrency or ordering assumptions sound?

#### Security

- Are there injection risks (SQL, shell, HTML, path traversal)?
- Is user input validated and sanitized at system boundaries?
- Are secrets, tokens, or credentials kept out of code and logs?
- Are authentication and authorization checks correct and complete?
- Are dependencies introduced without known vulnerabilities?

#### Performance

- Are there N+1 query patterns, unbounded loops, or unnecessary allocations?
- Are expensive operations cached or deferred where appropriate?
- Does the change affect hot paths in a way that warrants profiling?

#### Test coverage

- Are new behaviors covered by tests?
- Do tests exercise error paths and edge cases, not just the happy path?
- Are existing tests still meaningful after the change, or do they need updating?

#### Documentation

- Are public APIs, exported functions, and non-obvious logic commented?
- Is the PR description accurate and complete?
- Do `README`, `CHANGELOG`, or other docs need updating?

### 4. Format all feedback with format-review-comments

Apply the `format-review-comments` skill to every comment you write. Choose the label that matches the severity and intent:

| Situation | Recommended label |
|-----------|------------------|
| Must be fixed before merge | `issue [blocking]` |
| Strong recommendation | `suggestion` |
| Minor style or preference | `nitpick` |
| Something done well | `praise` |
| Needs clarification | `question` |
| Small unambiguous fix | `todo` |
| Informational only | `note` |

For the full label and decoration reference, read `../format-review-comments/references/conventional-comments-spec.md`.

### 5. Write a review summary

After the per-comment feedback, write a brief summary (3–7 sentences) covering:

- Overall assessment: is the PR ready to merge, needs minor changes, or needs significant rework?
- The most important finding (if any blocking issues exist, name them)
- Any patterns worth calling out across the diff
- Any dimensions that were skipped and why

Format the summary as a `note` conventional comment.

### 6. Refine prose

Apply the `refine-prose` skill to the review summary and all comments before posting. Run it silently — do not announce the refinement step.

## Examples

**Invocation with PR number:**
```
/review-pr 42
```

**Invocation with URL:**
```
/review-pr https://github.com/owner/repo/pull/42
```

**Sample output (logic finding):**
```
issue [blocking]: `processItems` dereferences `items[0]` before checking that the slice is non-empty — this will panic on an empty input

Add a length guard before the dereference:
if len(items) == 0 {
    return nil, ErrNoItems
}
```

**Sample output (positive finding):**
```
praise: clean separation between the HTTP layer and business logic — the new handler delegates immediately to the service and has no domain knowledge of its own
```

**Sample output (summary):**
```
note: overall this PR is close to merge-ready. One blocking nil-dereference must be fixed; the two suggestions and one nitpick are optional. Test coverage is thorough. Documentation dimension was skipped — no public API surface changed.
```
