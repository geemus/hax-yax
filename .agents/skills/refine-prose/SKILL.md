---
name: refine-prose
description: >
  Polishes drafted prose for clarity, conciseness, and consistent voice before
  it is posted or presented. Apply after drafting any plan, review summary, or
  written output to catch filler phrases, passive constructions, and
  inconsistent terminology. Use when the user asks to polish, clean up, improve,
  edit, or tighten writing — also triggered by "refine this draft", "make this
  clearer", or "improve my writing". Also composable: other skills invoke it as
  a late-stage refinement step.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.2"
---

# Refine Prose

Polishes a draft for clarity, conciseness, and consistent voice. Apply after producing any written output — a plan, review summary, comment, or document — before presenting or posting it.

## When to apply

- After drafting a plan, proposal, or structured document
- After writing a PR review summary or set of review comments
- Before presenting any multi-sentence output to a user
- Whenever explicitly invoked via `/refine-prose` on arbitrary text

When another skill calls it as a composable step, run silently and present only the refined result.

## Instructions

Work through each check in order. Apply fixes silently — do not narrate the edits.

### 1. Cut filler phrases

Remove words and phrases that add length without meaning. Common culprits:

| Remove | Replace with |
|--------|-------------|
| "in order to" | "to" |
| "due to the fact that" | "because" |
| "at this point in time" | "now" |
| "it is worth noting that" | _(delete or keep the fact)_ |
| "please note that" | _(delete; just state the fact)_ |
| "as mentioned above/below" | _(delete or use a direct reference)_ |
| "basically", "essentially", "literally" | _(delete unless they carry meaning)_ |
| "very", "really", "quite", "rather" | _(delete or strengthen the noun/verb)_ |

### 2. Prefer active voice

Rewrite passive constructions as active where the actor is known and the active form is clearer.

- "The tests were run by CI" → "CI ran the tests"
- "The config will be updated" → "Update the config" _(in instructions)_

Passive is acceptable when the actor is unknown, unimportant, or when convention demands it (e.g. "was introduced in v2").

### 3. Tighten sentences

- Break sentences longer than ~30 words into two unless the length is necessary for precision.
- Remove redundant restatements: if the same idea appears twice within a paragraph, keep the clearer instance.
- Ensure each sentence has one main idea.

### 4. Enforce parallel structure in lists

All items in a bulleted or numbered list must use the same grammatical form.

- If the first item starts with a verb ("Run the tests"), all items must start with a verb.
- If the first item is a noun phrase ("Test coverage"), all items must be noun phrases.

### 5. Standardize terminology

Within a single document, use one term for each concept consistently. If a term was introduced earlier, reuse it exactly — do not alternate between synonyms (e.g. do not mix "pull request" and "PR" in formal plans; pick one and use it throughout).

Default preferences:
- "pull request" in formal prose; "PR" is acceptable in informal or space-constrained contexts
- Oxford comma in lists of three or more items
- Sentence case for headings unless the document already uses title case throughout

### 6. Match formality to context

- **Plans and proposals**: formal, third-person where possible, no contractions
- **Review comments**: direct and specific; second person ("you") is appropriate; contractions acceptable
- **Instructions**: imperative mood, second person implicit ("Run tests", not "You should run tests")

## Composability

This skill is designed to be referenced by other skills as a late-stage refinement step. When wired in as a composable step, run it silently and present only the refined output — do not announce that you are running it.

Callers reference it with a line such as:

> Apply the `refine-prose` skill to the draft before presenting.
