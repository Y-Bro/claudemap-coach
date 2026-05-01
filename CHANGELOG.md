# Changelog

All notable changes to `claudemap-coach`. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows [SemVer](https://semver.org/).

## [Unreleased]

### Added
- `/claudemap-coach:create <topic>` — first working command. End-to-end flow: discovery questions → trend analysis (WebSearch) → draft generation → parallel review by `roadmap-reviewer` (structural) and `roadmap-specialist` (persona-injected domain expert) → critique-revise loop (max 2 iterations, auto-applies `critical`/`high` only) → user gating of `medium`/`low` suggestions → atomic write to `~/claudemap/<slug>.md` + JSON sidecar → `## Run Stats` block with token usage and approximate cost.
- `skills/roadmap-coach/SKILL.md` — methodology with create-mode workflow, invariants, error handling, and stubs for the other modes.
- `agents/roadmap-reviewer.md` — generic structural reviewer.
- `agents/roadmap-specialist.md` — persona-injected domain expert with WebSearch verification.
- Templates: `skills/roadmap-coach/templates/roadmap.md.tmpl` (markdown skeleton with embedded Mermaid) and `templates/progress.json` (sidecar JSON Schema).
- References: `skills/roadmap-coach/references/trend-search-patterns.md` (WebSearch query cookbook) and `references/model-pricing.md` (cost-calculation table).

### Notes
- `/claudemap-coach:update`, `:refresh`, and `:review` are not yet implemented and will return a "coming in a future release" message.
- Plugin validates with `claude plugin validate .`.

## [0.1.0] - 2026-05-01

### Added
- Plugin manifests (`.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`) — establishes plugin identity, author, license, and marketplace metadata.
