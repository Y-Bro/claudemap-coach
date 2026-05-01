---
description: Get a fresh expert review of an existing roadmap (read-only by default)
argument-hint: [path]
---

The user wants an expert review of an existing roadmap. **Read-only by default** — do not edit anything unless the user explicitly asks you to apply a specific suggestion.

**Path argument:** $ARGUMENTS

Invoke the `roadmap-coach` skill in **review** mode and follow its review-mode workflow exactly:

1. Locate the roadmap. If `$ARGUMENTS` is provided, use it. Otherwise ask the user where to look with three options: (1) current directory, (2) global library (`claudemapDir` if set, else `~/claudemap/`), (3) custom path. Then list roadmaps in the chosen directory and ask the user to pick.
2. Read and validate the sidecar JSON.
3. Use the persona stored in the sidecar (or derive a fresh one from the topic if missing).
4. Dispatch the two reviewers **in parallel from a single message** against the current roadmap. Use bare `subagent_type` values exactly: `"roadmap-reviewer"` and `"roadmap-specialist"` — never namespaced. If a subagent reports 0 tool uses, do NOT re-dispatch with a different name; see SKILL.md §5.
5. Consolidate feedback into one report grouped by severity, then by source.
6. Present the report to the user — **no edits applied**.
7. If the user follows up with `"apply suggestion #N"` (or similar), apply that single suggestion via atomic write to both the markdown and sidecar. Otherwise, leave the files untouched.
8. Append the `## Run Stats` block.

If `$ARGUMENTS` is empty and no roadmaps exist in the chosen directory, tell the user to run `/claudemap-coach:create` first or re-prompt for a different location.
