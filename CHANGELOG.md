# Changelog

All notable changes to this project are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/); versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased] — 2026-04-23

### Changed

- `implement-plan` v1.1: defer all lint, typecheck, and test runs until after `review-pr` completes, and consolidate them into a single dedicated final-validation step (new step 6) with a bounded fix-and-recommit loop (2-retry cap, then escalation). Eliminates redundant validation of intermediate per-task and per-review-fix states. PR-opening renumbered to step 7.

## [Unreleased] — 2026-04-09

### Changed

- Renamed repository from `geemus/skills` to `geemus/hax-yax`; updated plugin manifest name, repository URL, all `/skills:` namespace references to `/hax-yax:`, and example URLs across `README.md`, `AGENTS.md`, and `.agents/skills/upsert-plan/SKILL.md`
- Renamed `manage-plans` → `upsert-plan`: "upsert" more precisely describes the create-or-update operation; singular noun matches single-item-per-invocation behavior
- Renamed `manage-skills` → `upsert-skill`: same rationale — dominant operation is create-or-update a single skill
- Renamed `manage-sprites` → `manage-sprite`: singular noun correction; "manage" retained because destroy and exec are equal-weight operations alongside create/update
- Updated all cross-references in SKILL.md files, reference documents, AGENTS.md, and README.md to use the new names

## [1.0.0] — 2026-04-06

### Added

- Six production-ready skills: `create-commit`, `manage-plans`, `format-review-comments`, `manage-skills`, `refine-prose`, `review-pr`
- `.claude-plugin/plugin.json` manifest enabling installation via `claude --plugin-dir` and `claude plugin install`
- `README.md` with installation instructions and per-skill usage reference
- `.claude/skills` symlink for backward-compatible standalone Claude Code discovery
- Reference materials: `skill-format.md`, `conventional-comments-spec.md`, `secret-patterns.md`
