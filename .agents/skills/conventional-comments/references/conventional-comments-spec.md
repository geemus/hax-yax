# Conventional Comments — Full Specification

Source: [conventionalcomments.org](https://conventionalcomments.org/)

## Format

```
label [decorations]: subject

[body]

[discussion reference]
```

Every part after `label` is optional, but `subject` should almost always be present.

---

## Labels

### `praise`
Highlights something positive about the code or design. Never blocking. Use to acknowledge good work and encourage the author.

```
praise: elegant use of the iterator pattern here
```

### `nitpick`
A minor style, naming, or preference issue that does not affect correctness. Non-blocking by default. The author may address it at their discretion.

```
nitpick: variable name `tmp` could be more descriptive
```

### `suggestion`
Proposes a concrete improvement. Non-blocking by default; the author should consider it but is not required to act. Include enough detail for the author to evaluate the trade-off.

```
suggestion: consider caching this result — it's computed on every render

You could memoize with `useMemo` keyed on `userId`.
```

### `issue`
Points out a problem that must be addressed before merging. **Blocking by default.** Be specific about what is wrong and, where possible, how to fix it.

```
issue: this loop will run indefinitely if `items` is empty; add a guard or change the condition
```

### `todo`
A small, clearly scoped task the author should complete. Usually non-blocking unless decorated. Distinct from `issue` in that it is unambiguous and easy to execute.

```
todo: remove this debug log before merging
```

### `question`
Requests clarification without implying anything is wrong. Non-blocking. The author should answer but is not required to change code.

```
question: is this intentionally case-sensitive, or should we normalise to lowercase first?
```

### `thought`
Shares an idea or observation that doesn't warrant any action. Non-blocking. Use when you want to surface a consideration without creating pressure to act.

```
thought: if we ever need to support multiple currencies, this will need to change — not a concern now
```

### `chore`
Routine housekeeping: rename, reformat, move, update a comment. Non-blocking by default.

```
chore: update the docstring to reflect the new parameter name
```

### `note`
Provides context or information the author or reviewer may find useful. Non-blocking; no action required.

```
note: this mirrors the pattern used in `auth/middleware.go` — keeping it consistent is intentional
```

---

## Decorations

Decorations appear in brackets immediately after the label, comma-separated.

| Decoration | Meaning |
|------------|---------|
| `blocking` | Must be resolved before the PR can merge (overrides a label's default) |
| `non-blocking` | Does not need to be resolved before merging (overrides a label's default) |
| `if-minor` | Blocking only if the change required is small; author's call |
| `security` | Has security implications; treat with extra care |
| `performance` | Relates to performance or efficiency |
| `accessibility` | Relates to a11y concerns |

### Combining decorations

```
issue [non-blocking, if-minor]: variable shadowing — harmless here, but can confuse readers
```

```
suggestion [blocking, security]: user input is interpolated directly into the SQL query — use a parameterised query instead
```

---

## Blocking vs. non-blocking defaults

| Label | Default |
|-------|---------|
| `praise` | non-blocking |
| `nitpick` | non-blocking |
| `suggestion` | non-blocking |
| `issue` | **blocking** |
| `todo` | non-blocking |
| `question` | non-blocking |
| `thought` | non-blocking |
| `chore` | non-blocking |
| `note` | non-blocking |

Use `[blocking]` or `[non-blocking]` decorations to override the default.

---

## Tips

- Keep the **subject** to one line; move elaboration to the **body**.
- A comment without a label is ambiguous — always label.
- Prefer `suggestion` over `issue` when you are not certain something is wrong.
- Use `praise` freely; positive signals are as useful as critical ones.
- When a comment references an external discussion, link it at the end of the body.
