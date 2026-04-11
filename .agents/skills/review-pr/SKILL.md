---
name: review-pr
description: >
  Conducts systematic pull request reviews across logic, security, performance,
  test coverage, and documentation. Uses the format-review-comments skill to
  format all feedback. Use when the user asks to review a pull request, audit a
  diff, or give structured feedback on proposed code changes — also triggered by
  "look at this PR", "check PR #N", "can you review this", or "give feedback on
  this diff".
  TRIGGER when: user asks to review a pull request, audit a diff, or give
  feedback on proposed code changes.
  DO NOT TRIGGER when: user asks to review, evaluate, or audit a skill file
  (use review-skill instead).
license: Apache-2.0
metadata:
  author: geemus
  version: "1.3"
---

# Review PR

Conducts a structured, multi-dimensional review of a pull request and formats every comment using the `format-review-comments` skill.

## Instructions

### 1. Identify the PR

Accept the PR as:
- A GitHub PR URL (e.g. `https://github.com/owner/repo/pull/42`)
- A bare PR number (e.g. `42`) — infer the repo from `git remote get-url origin`
- A local diff or branch name — use `git diff <base>..<head>` to obtain the diff

If no PR is specified, ask the user: "Which PR would you like me to review? Provide a URL, PR number, or branch name."

Fetch the PR diff and description before proceeding.

### 2. Read the PR description

Understand the stated intent: what problem does the PR solve, what approach the author took, and what is explicitly out of scope. Use this as the lens for all review dimensions below.

### 3. Review each dimension

Work through every dimension in order. For each, read the relevant changed files and note findings. You do not need to comment on every dimension — skip a dimension only when the diff contains no changed lines that could affect it. Examples: skip Security when the only changes are to Markdown or CSS files; skip Performance when the only changes are to configuration or documentation; skip Test coverage when the PR is a documentation-only change.

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

**Output:** All review comments are written to the conversation only. To post them to the PR, pass the output to the appropriate GitHub tool or ask the user if they would like comments posted.

Apply the `format-review-comments` skill to every comment you write. Choose the label that matches the severity and intent:

| Situation | Recommended label |
|-----------|------------------|
| Must be fixed before merge | `issue [blocking]` |
| Strong recommendation | `suggestion` |
| Minor style or preference | `nitpick` |
| Needs clarification | `question` |
| Small unambiguous fix | `todo` |
| Informational only | `note` |

Do not write `praise` comments. Omit positive findings entirely — they add tokens without actionable content.

For the full label and decoration reference, read `../format-review-comments/references/conventional-comments-spec.md`.

### 5. Write a review summary

After the per-comment feedback, write a brief summary (3–7 sentences) covering:

- Overall assessment — ready to merge, needs minor changes, or needs significant rework
- The most important finding (name any blocking issues)
- Any patterns visible across the diff
- Any skipped dimensions and why

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

**Invocation with branch name:**
```
/review-pr feature/my-branch
```
The agent will run `git diff main..feature/my-branch` to obtain the diff.

**Sample output (logic finding):**
```
issue [blocking]: `processItems` dereferences `items[0]` before checking that the slice is non-empty — this will panic on an empty input

Add a length guard before the dereference:
if len(items) == 0 {
    return nil, ErrNoItems
}
```

**Sample output (suggestion with inline fix):**
```
suggestion: the new retry loop has no backoff — under sustained failures it will hammer the downstream service at full speed

Add exponential backoff:
delay := time.Second
for attempt := 0; attempt < maxRetries; attempt++ {
    if err := call(); err == nil { break }
    time.Sleep(delay)
    delay *= 2
}
```

**Sample output (summary):**
```
note: overall this PR is close to merge-ready. One blocking nil-dereference must be fixed; the two suggestions and one nitpick are optional. Test coverage is thorough. Documentation dimension was skipped — no public API surface changed.
```
