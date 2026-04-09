# Changelog

All notable changes to this project are documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/); versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased] — 2026-04-09

### Changed

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
