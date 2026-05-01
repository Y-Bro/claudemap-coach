# Changelog

All notable changes to `claudemap-coach`. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows [SemVer](https://semver.org/).

## [Unreleased]

### Added
- `/claudemap-coach:review [path]` — third working command. Read-only expert review of an existing roadmap. Dispatches `roadmap-reviewer` and `roadmap-specialist` in parallel against the current markdown, consolidates feedback into a single severity-grouped report, and presents it without editing. The user can opt-in to apply specific suggestions with `apply suggestion #N` (or `apply all critical/high`); otherwise no files change.
- `commands/roadmap-review.md` — slash command file.
- `skills/roadmap-coach/SKILL.md` §3 Review mode — full 8-step workflow (locate → load → resolve persona → parallel dispatch → consolidate → present read-only → optional apply → run stats).

### Changed
- `skills/roadmap-coach/SKILL.md` mode dispatch table — `review` is now `implemented (v0.4)`.
- Section numbering renumbered to insert §3 Review mode (downstream sections shifted by one).
- §7 stub list narrowed to `refresh` only.

### Notes
- `/claudemap-coach:refresh` is the only command still pending; it will land in a future release.

## [0.3.0] - 2026-05-01

### Added
- `/claudemap-coach:update [path]` — second working command. Walks through milestones one at a time, captures status changes (`done` / `in_progress` / `blocked` / `dropped` / `skip`) with optional notes, regenerates the markdown (checkboxes + Mermaid styling) from the sidecar, and atomic-writes both files. No subagents, no WebSearch — cheap operation. Supports shortcuts: `done all in phase {n}`, `skip phase {n}`, `quit`.
- `commands/roadmap-update.md` — slash command file.
- `skills/roadmap-coach/SKILL.md` §2 Update mode — full workflow (locate → load → walk → update sidecar → re-render → atomic write → summary + run stats).

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
