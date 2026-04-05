---
name: format-review-comments
description: >
  Formats code review and PR feedback using the Conventional Comments standard
  (label [decorations]: subject). Apply proactively whenever posting review
  comments or any critique of code or pull requests. Invoke directly when the
  user asks to format feedback, use conventional comments style, or label a
  review comment as blocking, suggestion, or nitpick.
license: Apache-2.0
metadata:
  author: geemus
  version: "1.3"
---

# Conventional Comments

Formats review comments using the [Conventional Comments](https://conventionalcomments.org/) standard, making intent unambiguous — readers know immediately whether a comment is blocking, optional, or informational.

## When to apply

Apply this format **proactively** whenever you:
- Post a PR review comment (inline or summary)
- Provide feedback on code, architecture, or a document
- Suggest a change or improvement in a review context

Use this format whenever you are reviewing anything — no explicit request needed.

## Format

```
label [decorations]: subject

[body]

[discussion reference]
```

- **label** — the comment's intent (see reference for full list)
- **decorations** — optional modifiers in brackets, comma-separated (e.g. `[blocking, if-minor]`)
- **subject** — a short summary; the main message
- **body** — optional elaboration, reasoning, or examples
- **discussion reference** — optional link to a related issue, PR, or discussion

## Core labels

| Label | Intent |
|-------|--------|
| `praise` | Highlights something positive; never blocking |
| `nitpick` | Minor style/preference issue; non-blocking by default |
| `suggestion` | Proposes an improvement; non-blocking by default |
| `issue` | Points out a problem that must be addressed; blocking by default |
| `todo` | Small, unambiguous task the author should complete |
| `question` | Asks for clarification; non-blocking |
| `thought` | Shares an idea that doesn't warrant action |
| `chore` | Simple housekeeping (rename, move, reformat); non-blocking |
| `note` | Informational; no action required |

For the full label and decoration reference, read `references/conventional-comments-spec.md`.

## Quick examples

```
praise: clean abstraction here — easy to follow

suggestion: consider extracting this into a helper so it can be reused

issue [blocking]: this will panic on nil input; add a nil check before dereferencing

question: why is this sorted descending? just want to make sure it's intentional

nitpick: s/recieve/receive/
```

