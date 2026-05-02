# Review AGENTS.md — Per-Dimension Checklist

Reference this checklist during Step 3 of the `review-agents-md` skill. Work through each dimension in order. Mark each item as a finding only when it is missing, incorrect, or unclear.

---

## Dimension 1: Completeness

Does the file cover every topic an agent needs to operate correctly?

- [ ] **Repository purpose** — states the repo's goal and domain explicitly
- [ ] **Directory structure** — documents the layout with enough detail for navigation
- [ ] **Coding conventions** — names the language(s), formatting rules, and style constraints
- [ ] **Git workflow** — covers branch naming, commit message format, and PR process
- [ ] **Available tools and scripts** — lists CLIs, scripts, and external tools an agent may use
- [ ] **Security constraints** — documents secrets management rules, destructive-operation guards, and any off-limits actions
- [ ] **Environment / setup** — mentions required dependencies or environment variables where applicable

**Severity guidance:**
- Missing section that would cause unsafe or incorrect agent behavior → `issue [blocking]`
- Missing section that would cause suboptimal but recoverable behavior → `suggestion`

---

## Dimension 2: Accuracy (cross-referencing)

Does every reference in the file point to something that actually exists?

### File and directory paths
- [ ] Extract all paths mentioned (absolute and relative)
- [ ] For each path, run `Glob` or `ls` to confirm it exists
- [ ] Flag clearly wrong paths as `issue`; flag ambiguous paths (generated, external) as `question`

### Branch names
- [ ] Identify any branch names stated (default branch, example feature branches)
- [ ] Run `git branch -a` to verify they exist
- [ ] Flag mismatches as `suggestion`

### Tool and CLI names
- [ ] Extract every tool, command, or script name referenced
- [ ] Run `which <tool>` or check PATH availability
- [ ] Surface failures as `question` (tool may exist in user's environment but not agent's)

### Skill and internal cross-references
- [ ] For any skill or file referenced by path, confirm the path exists with `Glob`
- [ ] Flag stale references as `issue`

**Severity guidance:**
- Path clearly does not exist and is not external → `issue`
- Path may be external, generated, or environment-specific → `question`
- Branch name mismatch → `suggestion`
- Tool not found locally → `question`

---

## Dimension 3: Agent-friendliness

Can an agent follow the instructions without human interpretation?

- [ ] **Explicit rules** — conventions are specific, not vague ("use 2-space indentation", not "follow good style")
- [ ] **Conditional branches** — if/else paths are spelled out, not implied
- [ ] **Explicit prohibitions** — "do not" statements name the action and why, not just imply restriction
- [ ] **Examples for non-obvious conventions** — complex rules have at least one example
- [ ] **Imperative mood** — instructions direct the agent ("run X", "use Y") rather than describe ("X is preferred")
- [ ] **No ambiguous sections** — no passage where two agents would make different decisions

**Severity guidance:**
- Ambiguity that could cause unsafe or conflicting behavior → `issue [blocking]`
- Vague guidance that reduces consistency → `suggestion`
- Style or phrasing that could be clearer → `nitpick`

---

## Dimension 4: Freshness

Does the documented state match the actual repository?

- [ ] **Directory structure** — compare documented layout against repo via `Glob`; flag discrepancies
- [ ] **Skills table** (if present) — verify every listed skill directory exists; flag missing directories as `issue`, missing entries as `suggestion`
- [ ] **Tool / script list** — verify listed tools still exist or are still relevant
- [ ] **Removed or renamed sections** — check for references to paths or conventions that no longer apply

**Severity guidance:**
- Documented item does not exist in repo → `issue`
- Item exists in repo but is not documented → `suggestion`

---

## Dimension 5: Scope coherence

Is the file focused on agent guidance?

- [ ] **No README duplication** — content that belongs in `README.md` (installation, marketing, API docs) is not duplicated here without agent-specific framing
- [ ] **No CHANGELOG content** — version history or release notes do not appear here
- [ ] **Single audience** — the file addresses agent behavior, not human onboarding in parallel
- [ ] **Appropriate depth** — sections are detailed enough for agent use but not padded with background that agents do not need

**Severity guidance:**
- Content that actively misleads an agent → `issue [blocking]`
- Content that belongs in another file → `suggestion`
- Redundant or padded content → `nitpick`

---

## Dimension 6: Leanness

Does the file avoid duplicating information already present in source-level documentation?

> *Heuristic: if a future agent could find this information by reading the source, it should not also appear in full here. Replace with a pointer.*

### Detect duplication with source documentation

- [ ] **Identify candidate sections** — locate sections that describe specific code units (modules, packages, classes, functions, types, callbacks, structs, byte-level layouts, error codes, symbol tables, or API surfaces)
- [ ] **Locate the authoritative source** — for each candidate, find the corresponding source file (use `Glob`/`Grep`, then `Read` the file's docstring, doc comment, header comment, or language-native annotation such as JSDoc, Rustdoc, Godoc, `@moduledoc`)
- [ ] **Compare wording** — check whether the AGENTS.md prose repeats the source documentation verbatim, near-verbatim, or only paraphrases the same facts
- [ ] **Check for implementation details** — flag content that belongs in the source itself (callback contracts, struct field tables, byte-level schemas, exhaustive symbol enumerations) rather than the agent instructions

### Pointer test

For each candidate section, ask: *Could this section be replaced with a one-line pointer to the authoritative source without losing agent-usable information?* If yes, the section is a leanness finding.

A "pointer" looks like:

```
See the doc comment for `Foo` at `src/foo.ts`.
```

### Severity guidance

- **`suggestion`** — *pointer preferred*: section could be a one-line pointer to source documentation, but the full prose is defensible (e.g. provides agent-specific framing, summarizes for quick reference)
- **`issue`** — *verbatim duplicate*: section repeats source documentation verbatim or near-verbatim, creating two places to maintain the same information
- **`issue [blocking]`** — *bloat impairs usability* (rare): the file has grown so large from accumulated duplication that an agent struggles to locate actionable guidance

### What is not a leanness finding

- Brief summaries that orient an agent before pointing to the source (these aid navigation)
- Cross-cutting conventions that span multiple files (these have no single authoritative source to point to)
- Workflow guidance that happens to reference symbol names (the focus is on the workflow, not on duplicating the source's own documentation)
