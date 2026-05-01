---
description: Bring an existing roadmap up to date with the latest trends, tools, and resources
argument-hint: [path]
---

The user wants to refresh an existing roadmap against current industry trends.

**Path argument:** $ARGUMENTS

Invoke the `roadmap-coach` skill in **refresh** mode and follow its refresh-mode workflow exactly:

1. Locate the roadmap. If `$ARGUMENTS` is provided, use it. Otherwise ask the user where to look with three options: (1) current directory, (2) global library (`claudemapDir` if set, else `~/claudemap/`), (3) custom path. Then list roadmaps in the chosen directory and ask the user to pick.
2. Read and validate the sidecar JSON.
3. Resolve the persona (reuse from sidecar; derive fresh if missing).
4. Run trend re-analysis via WebSearch, informed by the sidecar's `lastRefreshed` date — emphasize what's changed since then. Use refresh-specific patterns from `references/trend-search-patterns.md`.
5. Diff fresh signals against the sidecar's `trendSignals` baseline. Identify newly-emerged items, newly-deprecated items, link rot, and updated salary anchors.
6. Generate change proposals — each tied to a specific location in the roadmap with reasoning and evidence URL.
7. User gates each proposal: `[accept | revise | reject]`.
8. Apply accepted changes to produce a refreshed draft.
9. Dispatch `roadmap-reviewer` and `roadmap-specialist` in parallel on the refreshed draft. Run the critique-revise loop (max 2 iterations; auto-apply only `critical`/`high`).
10. Surface remaining `medium`/`low` review items for user gating.
11. Atomic-write the refreshed markdown + sidecar. Bump `sidecar.lastRefreshed`; update `sidecar.trendSignals` with the fresh baseline.
12. Append the `## Run Stats` block.

If `$ARGUMENTS` is empty and no roadmaps exist in the chosen directory, tell the user to run `/claudemap-coach:create` first or re-prompt for a different location.
