---
description: Walk through your roadmap and update milestone progress
argument-hint: [path]
---

The user wants to update milestone progress on an existing roadmap.

**Path argument:** $ARGUMENTS

Invoke the `roadmap-coach` skill in **update** mode and follow its update-mode workflow exactly:

1. Locate the roadmap. If `$ARGUMENTS` is provided, use it. Otherwise ask the user where to look with three options: (1) current directory, (2) global library (`claudemapDir` if set, else `~/claudemap/`), (3) custom path. Then list roadmaps in the chosen directory and ask the user to pick.
2. Read and validate the sidecar JSON (source of truth).
3. Walk milestones one at a time. For each non-terminal milestone, ask for new status: `[done | in_progress | blocked | dropped | skip]`. Capture optional notes for `blocked` and `dropped`.
4. Update the sidecar (statuses, `completedAt` timestamps, `lastUpdated`).
5. Re-render the markdown from the sidecar (checkboxes, Mermaid styling).
6. Atomic-write sidecar + markdown.
7. Print a completion summary and the `## Run Stats` block.

If `$ARGUMENTS` is empty and no roadmaps exist in the chosen directory, tell the user to run `/claudemap-coach:create` first or re-prompt for a different location.
