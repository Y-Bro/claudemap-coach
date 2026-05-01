# Changelog

All notable changes to `claudemap-coach`. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows [SemVer](https://semver.org/).

## [Unreleased]

### Added
- `/claudemap-coach:update [path]` — second working command. Walks through milestones one at a time, captures status changes (`done` / `in_progress` / `blocked` / `dropped` / `skip`) with optional notes, regenerates the markdown (checkboxes + Mermaid styling) from the sidecar, and atomic-writes both files. No subagents, no WebSearch — cheap operation. Supports shortcuts: `done all in phase {n}`, `skip phase {n}`, `quit`.
- `commands/roadmap-update.md` — slash command file.
- `skills/roadmap-coach/SKILL.md` §2 Update mode — full workflow (locate → load → walk → update sidecar → re-render → atomic write → summary + run stats).

### Changed
- `skills/roadmap-coach/SKILL.md` mode dispatch table — `update` is now `implemented (v0.3)`.
- Section numbering renumbered to insert §2 Update mode (downstream sections shifted by one).
- §6 stub list narrowed to `refresh` and `review` only.

### Notes
- `/claudemap-coach:refresh` and `/claudemap-coach:review` are still not implemented and will return a "coming in a future release" message.

## [0.2.0] - 2026-05-01

### Added
- `/claudemap-coach:create <topic>` — first working command. End-to-end flow: discovery questions → trend analysis (WebSearch) → draft generation → parallel review by `roadmap-reviewer` (structural) and `roadmap-specialist` (persona-injected domain expert) → critique-revise loop (max 2 iterations, auto-applies `critical`/`high` only) → user gating of `medium`/`low` suggestions → atomic write to `~/claudemap/<slug>.md` + JSON sidecar → `## Run Stats` block with token usage and approximate cost.
- `skills/roadmap-coach/SKILL.md` — methodology with create-mode workflow, invariants, error handling, and stubs for the other modes.
- `agents/roadmap-reviewer.md` — generic structural reviewer.
- `agents/roadmap-specialist.md` — persona-injected domain expert with WebSearch verification.
- Templates: `skills/roadmap-coach/templates/roadmap.md.tmpl` (markdown skeleton with embedded Mermaid) and `templates/progress.json` (sidecar JSON Schema).
- References: `skills/roadmap-coach/references/trend-search-patterns.md` (WebSearch query cookbook) and `references/model-pricing.md` (cost-calculation table).

## [0.1.0] - 2026-05-01

### Added
- Plugin manifests (`.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`) — establishes plugin identity, author, license, and marketplace metadata.
