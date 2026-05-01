# Changelog

All notable changes to `claudemap-coach`. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows [SemVer](https://semver.org/).

## [Unreleased]

_No unreleased changes yet._

## [0.5.0] - 2026-05-01

First soft-launch release. All four commands functional, save-location UX polished, repo readied for public distribution.

### Added
- **Save-location prompt** across all four commands. `create` now asks at the start where to write the new roadmap with three options: current directory, global library (`claudemapDir` if set, else `~/claudemap/`), or a custom path. `update` / `review` / `refresh` ask the same three-option question to decide where to look for an existing roadmap. Explicit `$ARGUMENTS` paths bypass the prompt. Resolves the surprise of roadmaps silently landing in `~/claudemap/` instead of the user's working repo.
- `/claudemap-coach:refresh [path]` — fourth working command. Brings an existing roadmap up to date against current trends. Loads sidecar baseline → re-runs trend analysis via WebSearch (informed by `lastRefreshed`) → diffs fresh signals against baseline → generates change proposals (link rot, newly emerged, newly deprecated, salary drift) → user gates each proposal → applies accepted changes → re-runs `roadmap-reviewer` + `roadmap-specialist` in parallel with the same critique-revise loop as create (cap 2, auto-apply `critical`/`high`) → user gates remaining `medium`/`low` → atomic-writes both files → bumps `lastRefreshed` and updates `trendSignals` baseline.
- `commands/roadmap-refresh.md` — slash command file.
- `skills/roadmap-coach/SKILL.md` §4 Refresh mode — full 12-step workflow.
- `CONTRIBUTING.md`, `SECURITY.md`, GitHub issue + PR templates — community scaffolding for public distribution.

### Changed
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` — version bumped to `0.5.0`; marketplace entry now includes `homepage`, `license`, and `keywords`.
- `skills/roadmap-coach/SKILL.md` §1 Create mode — inserted Step 1.5 (save-location prompt); Step 9 now reads the chosen directory instead of computing `~/claudemap/<slug>.md` directly.
- `skills/roadmap-coach/SKILL.md` §2/§3/§4 — Step 1 of each mode now runs the same three-option location prompt before listing roadmaps.
- `skills/roadmap-coach/SKILL.md` mode dispatch table — `refresh` is now `implemented (v0.5)`. **All four commands now functional.**
- `README.md` — dropped pre-release status banner and "(planned)" license caveat now that the LICENSE file is real MIT and all four commands ship.
- Section numbering in SKILL.md: inserted §4 Refresh; renumbered downstream invariants/error/edge sections; removed the now-obsolete "Other modes (not yet released)" stub section entirely.

### Notes
- Feature parity with the design is complete. Validation against real users is the next milestone before cutting `1.0.0`.
- Known gaps tracked for `0.6.0`+: lint script, automated test fixtures, end-to-end checklist, sample roadmaps in `examples/`.

## [0.4.0] - 2026-05-01

### Added
- `/claudemap-coach:review [path]` — third working command. Read-only expert review of an existing roadmap. Dispatches `roadmap-reviewer` and `roadmap-specialist` in parallel against the current markdown, consolidates feedback into a single severity-grouped report, and presents it without editing. The user can opt-in to apply specific suggestions with `apply suggestion #N` (or `apply all critical/high`); otherwise no files change.
- `commands/roadmap-review.md` — slash command file.
- `skills/roadmap-coach/SKILL.md` §3 Review mode — full 8-step workflow (locate → load → resolve persona → parallel dispatch → consolidate → present read-only → optional apply → run stats).

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
