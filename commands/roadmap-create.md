---
description: Start a new career or learning roadmap with claudemap-coach
argument-hint: <topic>
---

The user wants to create a new career or learning roadmap.

**Topic from user:** $ARGUMENTS

Invoke the `roadmap-coach` skill in **create** mode and follow its create-mode workflow exactly:

1. Validate the topic (ask for clarification if vague; decline politely if out of scope per the skill's scope rules).
2. **Save-location prompt** — ask the user where to save the roadmap with three options: (1) current directory, (2) global library (`claudemapDir` if set, else `~/claudemap/`), (3) custom path. Skip this prompt only if `$ARGUMENTS` already includes an explicit output path.
3. Run the discovery script — ask the questions one at a time, never multi-question.
4. Run trend analysis via WebSearch using the patterns in `references/trend-search-patterns.md`.
5. Render a draft roadmap using `templates/roadmap.md.tmpl`.
6. Derive a specialist persona from the topic and target.
7. Dispatch `roadmap-reviewer` and `roadmap-specialist` subagents in parallel (single message, two `Agent` tool blocks).
8. Run the critique-revise loop (max 2 iterations; auto-apply only `critical` and `high`; surface `medium`/`low` to the user at the end).
9. Atomic-write the roadmap markdown and JSON sidecar to the directory chosen at step 2, as `<slug>.md` and `<slug>.json`.
10. Append a `## Run Stats` block to the terminal output with token usage and approximate cost.

If the user did not provide a topic in `$ARGUMENTS`, ask them what roadmap they want to create before invoking the skill.
