# Contributing to claudemap-coach

Thanks for your interest! This is a small, single-maintainer project, so the contribution flow is intentionally lightweight.

## Ways to contribute

- **Report bugs** — open an issue with the [Bug report](.github/ISSUE_TEMPLATE/bug_report.md) template. Steps to reproduce + the run-stats block from the failed run are gold.
- **Request features** — open an issue with the [Feature request](.github/ISSUE_TEMPLATE/feature_request.md) template. Lead with the problem you're trying to solve, not the solution you have in mind.
- **Submit roadmap examples** — if `/claudemap-coach:create` produced a great roadmap and you're willing to share it, open a PR adding it under `examples/<slug>.md`. Strip anything personal.
- **Improve the prompts and skill files** — most of this repo is markdown that drives Claude. Edits to `skills/roadmap-coach/SKILL.md`, the templates, or the agent personas are very welcome.

## Project layout

```
.claude-plugin/      Plugin manifests (plugin.json, marketplace.json)
agents/              Subagent definitions (roadmap-reviewer, roadmap-specialist)
commands/            Slash command files (one per /claudemap-coach:* command)
skills/roadmap-coach/
  SKILL.md             Methodology — the source of truth for all four modes
  templates/           Roadmap markdown skeleton + sidecar JSON schema
  references/          WebSearch query patterns + model-pricing lookup
CHANGELOG.md         Keep-a-Changelog format, SemVer
LICENSE              MIT
```

## Development setup

There is no build step. The plugin is markdown + JSON.

To work on it locally:

1. Clone the repo.
2. From the repo root, install it into your Claude Code as a local plugin (see Claude Code's plugin development docs for the current command — typically a `--source <path>` form of `claude plugin install`).
3. Run any of the four commands and iterate.

To validate JSON sidecars manually, use the schema at `skills/roadmap-coach/templates/progress.json`.

## Branch and commit conventions

- One feature per branch, branched from `main`.
- Branch names: `feat/<slug>` for features, `fix/<slug>` for bug fixes, `docs/<slug>` for docs-only.
- Push every commit to the remote — feel free to push WIP, it's just a backup.
- Merge to `main` via PR with `--no-ff` so the merge commit is preserved.
- Commit messages: short imperative subject line (under 72 chars), optional body explaining the **why**. No required prefix.

## Versioning

SemVer. Pre-`1.0.0`, minor bumps may include breaking changes if needed; we'll note them prominently in the CHANGELOG. Once we ship `1.0.0`, breaking changes require a major bump.

## Code review

There's no formal review process for a single-maintainer project — PRs are reviewed by Bharath. Larger changes (new commands, new modes, schema changes) should open a discussion issue first to avoid wasted work.

## Questions?

Open an issue tagged `question`, or reach out via the email in `.claude-plugin/plugin.json`.
