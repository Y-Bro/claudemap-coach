---
name: roadmap-coach
description: Conversational coach that turns career and learning goals into trend-aware, expert-reviewed roadmaps with embedded Mermaid timelines. Use when the user wants to plan a career transition, salary jump, role change, certification path, or skill mastery.
---

# Roadmap Coach

This skill is invoked by the four `/claudemap-coach:*` commands. The command sets a **mode**: `create`, `update`, `refresh`, or `review`.

## Mode dispatch

| Mode    | Status                | Section                         |
|---------|-----------------------|---------------------------------|
| create  | implemented (v0.2)    | §1 Create mode                  |
| update  | implemented (v0.3)    | §2 Update mode                  |
| refresh | not yet released      | §6 Stub                         |
| review  | not yet released      | §6 Stub                         |

---

## §1 Create mode

The end-to-end `create` workflow has nine ordered steps. Do them in order. **Do not skip steps. Do not combine steps that are listed separately.**

### Step 1 — Validate the topic

The command passes a topic string. Decide:

- **Out of scope?** If the topic is clearly outside career or learning (life-coaching, fitness, business launch, finance), decline politely and suggest rephrasing as a learning goal if applicable. Stop.
- **Too broad?** ("become a billionaire", "be successful") — flag and ask for a narrower target. Stop until clarified.
- **Too vague?** ("grow", "improve") — ask one clarifying question. Don't proceed until topic is concrete enough to drive trend search.
- **In scope and concrete?** Proceed to Step 2.

### Step 2 — Discovery script

Ask up to 7 questions, **one at a time**, in this order. Skip questions whose answers the user already provided in their topic prompt. The first 4 are mandatory.

1. **Current state** (mandatory) — role, level, key skills.
2. **Concrete target** (mandatory if not in topic) — role / outcome / specific number.
3. **Timeline** (mandatory) — months or years.
4. **Hours/week available** (mandatory) — realistic budget for execution.
5. **Budget for paid resources** — skip if topic implies free-only ("self-taught", "no budget").
6. **Hard constraints** — full-time job, family, location, accessibility. Skip if user says "no constraints."
7. **Learning style preference** — books, courses, projects, mentors, mix. Skip if topic strongly implies one.

**Never ask multiple questions in one message.** Wait for an answer before asking the next.

### Step 3 — Trend analysis

Once discovery is complete, run WebSearch using the patterns in `references/trend-search-patterns.md`. Pick 4–6 query categories most relevant to the topic and target. Batch them as **parallel WebSearch calls in a single message**.

Hard cap: **8 WebSearch calls per `create` invocation** (the specialist gets its own 5-call budget later).

Extract structured signals from the results:
- Top in-demand skills (frequency-ranked).
- Recommended resources with URLs (deduped; prefer reputable domains — see the patterns file for the allow-list).
- Salary anchors if applicable.
- Deprecated or declining items (warnings).

Save these signals; you'll write them into the sidecar's `trendSignals` field at Step 8.

### Step 4 — Persona derivation

Derive a specialist persona string from the topic + target. Persona must be specific enough to give the specialist a real perspective.

**Examples:**
- topic: "AI Engineer in 1 year" → persona: "Senior ML Engineer + hiring-committee member at a top AI lab in the present year, focused on candidates from non-AI backgrounds."
- topic: "20 LPA → 50 LPA SWE in India" → persona: "Indian engineering director with promotion-committee experience at top-tier tech companies, familiar with the FAANG-India and Indian-startup compensation ladders."
- topic: "Master Rust async" → persona: "Senior Rust systems engineer with deep expertise in tokio, lifetimes, and production async deployment."

**Fallback if persona generation fails:** "Senior practitioner in {topic} with hiring/mentorship experience." Log the fallback in the run stats.

### Step 5 — Draft generation

Render the roadmap using `templates/roadmap.md.tmpl`. Fill in:
- Header (topic, generated_date, model_id).
- Overview (current state, target, timeline, hours/week, budget, constraints).
- Phases (typically 3–6, scaled to timeline). Each phase has: title, date range, milestones with resource links, success criterion.
- Resources by category section.
- Mermaid gantt diagram (phases as sections).
- Footer with persona used and trend signals summary.

**Resource link discipline:** every named resource MUST have a real URL pulled from the trend-analysis results, with a `verified-{YYYY-MM}` tag. **No fabricated paths.** If you don't have a real URL for a resource, omit the resource — don't invent one.

### Step 6 — Parallel review dispatch

Dispatch `roadmap-reviewer` and `roadmap-specialist` subagents **in parallel from a single message** (two `Agent` tool blocks in the same response, not sequential calls).

- **roadmap-reviewer** prompt: include the full draft roadmap and ask for structural review. No persona — generic checks only.
- **roadmap-specialist** prompt: include the full draft roadmap **and** the persona string from Step 4. Instruct it to fully adopt the persona and use WebSearch to verify currency.

Each subagent returns a JSON list of `{location, severity, issue, suggestion|recommendation, evidence_url?}` items where `severity ∈ {critical, high, medium, low}`.

### Step 7 — Critique-revise loop

```
loop_iter = 0
while loop_iter < 2:
    if both subagents returned no `critical` and no `high` items:
        break
    auto_apply_all_critical_and_high_fixes(reviewer_output, specialist_output)
    log each auto-applied fix to revisionLog: {iteration: loop_iter, source, original, revised, reason}
    re-dispatch reviewer + specialist in parallel on the revised draft
    loop_iter += 1
```

**Guardrails:**
- Auto-apply scope: only `critical` and `high`. Never auto-apply `medium` or `low`.
- If the cap is hit and reviewers still flag `critical`/`high`, surface them prominently to the user with: "Reviewers couldn't agree on these in 2 rounds — you decide." **Do not auto-write.** Wait for user arbitration.
- If reviewer and specialist propose **conflicting** `critical`/`high` fixes for the same location: surface the conflict to the user mid-loop instead of auto-resolving.

### Step 8 — User gating of medium/low

After the loop completes (convergence or cap), present the **final draft** plus the accumulated **revision log** plus the **remaining `medium`/`low` suggestions** to the user.

For each `medium`/`low` suggestion, ask: `[accept | revise | reject]`. Apply user decisions.

### Step 9 — Atomic write + run stats

Determine the output path:
- Default: `~/claudemap/<slug>.md` where slug is derived from the topic (lowercase, hyphenated).
- Override: if user passed an explicit path, use it. If `claudemapDir` plugin setting is set, use that as the directory.

**Atomic write pattern:**
1. Stage both files: write to `<path>.md.tmp` and `<path>.json.tmp`.
2. Validate:
   - Sidecar JSON parses against `templates/progress.json` schema.
   - Markdown contains the required headings (Overview, Phases, Resources by category, Timeline).
   - All Mermaid blocks parse (regex check on opening/closing fences + at least one node line).
3. Two-step rename: rename sidecar `.tmp` → final, then markdown `.tmp` → final. (Not POSIX-atomic across two files, but recoverable: if step 2 fails, the sidecar is already correct and can be used to regenerate the markdown.)
4. If any validation fails: keep originals, surface error, don't touch real files.

**Roadmap clobber:** if `<path>.md` already exists, prompt: "Roadmap exists. Overwrite, version-bump (`<slug>-v2.md`), or cancel?"

**Concurrent runs:** sidecar gets a `lockedAt` timestamp during write. If a second command sees a recent lock (<5 min), prompt to override or wait.

**Run stats output** — append to terminal (not the saved file):

```
## Run Stats
- Input tokens:        X (cache reads: A, fresh: B)
- Output tokens:       Y
- Cache writes:        C
- Subagent runs:       2 × N iterations
- WebSearch calls:     M
- Approx cost:         $0.XX  (model: <model_id>, prices verified <YYYY-MM>)
```

Mechanism for collecting stats:
1. At command start, record current session transcript path (`~/.claude/projects/<project>/<session>.jsonl`) and byte offset.
2. At command end, re-read transcript from that offset, tally `message.usage.{input_tokens, output_tokens, cache_read_input_tokens, cache_creation_input_tokens}` across assistant messages. Subagent usage is included in parent's tool-result messages.
3. Compute cost from `references/model-pricing.md`.
4. If transcript file unavailable (sandbox / unusual harness), output: `"Token reporting unavailable in this environment"`.

---

## §2 Update mode

The `update` workflow is much cheaper than `create` — no subagents, no WebSearch, no critique loop. Just a guided walk through the existing milestones with sidecar + markdown regeneration.

### Step 1 — Locate the roadmap

- If the command passed an explicit path → use it.
- Otherwise: list `*.md` files in the default roadmap directory (the `claudemapDir` plugin setting, or `~/claudemap/` as fallback). For each, show: filename, topic (from sidecar), last-updated date, completion percentage. Ask the user to pick one.
- If no roadmaps exist, tell the user: `"No roadmaps found in <dir>. Run /claudemap-coach:create <topic> first."` Stop.

### Step 2 — Load and validate

- Read the sidecar JSON (`<path>.json` next to `<path>.md`). The sidecar is the source of truth.
- If the sidecar is missing: tell the user, offer no recovery — they must regenerate via `/claudemap-coach:create`.
- Validate against the schema in `templates/progress.json`. If `schemaVersion` doesn't match `1.0`, refuse and surface the version mismatch (migration tooling lands later).

### Step 3 — Walk milestones

Iterate through phases in order. For each phase, iterate through milestones.

For each milestone whose current status is **not** `done` or `dropped`, ask:

```
Phase {n}, milestone {m}: "{milestone text}"
Current status: {current_status}
New status? [done | in_progress | blocked | dropped | skip]
```

- `done` — set `status: "done"`, set `completedAt` to current ISO 8601 timestamp.
- `in_progress` — set `status: "in_progress"`. Optionally ask for a one-line note.
- `blocked` — set `status: "blocked"`. Ask for a one-line note explaining what's blocking.
- `dropped` — set `status: "dropped"`. Ask for a one-line note explaining why.
- `skip` — leave the milestone untouched and continue.

**Shortcuts the user can type instead of a status:**
- `done all in phase {n}` — mark every remaining milestone in that phase as `done`.
- `skip phase {n}` — skip the rest of that phase and move on.
- `quit` — stop walking; save what's been set so far.

### Step 4 — Update sidecar

- Apply all status changes and notes.
- Update `lastUpdated` to current ISO 8601 timestamp.
- Set `lockedAt` during write; clear on success.

### Step 5 — Re-render markdown

Regenerate the markdown body from the sidecar. Specifically:

- **Checkboxes** in milestone lists:
  - `pending` → `- [ ] {text}`
  - `in_progress` → `- [ ] 🚧 {text}` (with note appended in italics if present)
  - `done` → `- [x] {text}`
  - `blocked` → `- [ ] 🚫 {text} — _blocked: {note}_`
  - `dropped` → `- [ ] ~~{text}~~ — _dropped: {note}_`

- **Mermaid gantt** — phase status markers reflect completion:
  - All milestones in a phase done → `:done, p{n}, ...`
  - Some milestones in a phase done, others not → `:active, p{n}, ...`
  - No milestones in a phase done → leave default

- Preserve everything outside the `## Phases` and `## Timeline` sections (Overview, Resources by category, Specialist persona, Trend signals, Sources). Update mode does not touch those.

### Step 6 — Atomic write

Use the same atomic-write pattern as create (Step 9 of §1):
1. Stage `<path>.md.tmp` and `<path>.json.tmp`.
2. Validate (sidecar against schema, markdown contains required headings, Mermaid blocks parse).
3. Two-step rename.
4. On any failure, keep originals untouched.

### Step 7 — Summary + run stats

Print a short completion summary, then the standard `## Run Stats` block:

```
Updated: N marked done, M in progress, B blocked, D dropped.
Phase completion: Phase 1: 80%, Phase 2: 30%, ...
Overall: P% complete.

## Run Stats
- Input tokens:        ...
- Output tokens:       ...
- ...
- Approx cost:         $0.XX  (model: <model_id>, prices verified <YYYY-MM>)
```

### Update-mode-specific invariants

- **Cheap operation** — no subagents, no WebSearch, no critique loop.
- **One milestone at a time** during the walk (the documented shortcuts are the only batching).
- **Sidecar is the source of truth** — markdown is regenerated, not patched in place.

---

## §3 Invariants (apply to all modes)

These rules are non-negotiable across every mode:

- **One question at a time** during user-facing dialogue. No multi-question messages.
- **WebSearch always before generation** in `create` and `refresh`. Trends ground the output.
- **Subagents always parallel** when dispatching multiple — single message, multiple `Agent` blocks.
- **Never finalize** without the critique-revise loop reaching convergence-or-cap AND user gating remaining `medium`/`low`.
- **Atomic write only** — never write the markdown without the sidecar, or vice versa.
- **Run stats always at end** of every command's terminal output.
- **No fabricated URLs.** Every resource link must come from a WebSearch result.

---

## §4 Error handling matrix

| Failure | Behavior |
|---|---|
| WebSearch returns no results | Continue with notice: `"Trend signals unavailable; roadmap based on model knowledge alone — recommend running /refresh later"`. Do NOT block. |
| WebSearch rate-limited | Back off, retry once after 30s. If still failing, degrade as above. |
| Subagent dispatch fails | Retry once. If still failing, fall back to single-agent self-review (lower quality, noted in run stats). |
| Reviewers disagree across iterations (cap hit, both still flag critical) | Surface both reviewers' final positions, mark `unresolved`, let user arbitrate. Do NOT auto-write. |
| User abandons mid-conversation | No partial roadmap written. Sidecar not created. Clean exit. |
| Persona generation fails | Fall back to generic persona (see Step 4). Log in run stats. |
| Write validation fails | Keep originals. Surface the validation error. Do not retry silently. |
| Stale `.tmp` file from prior run | Prompt: `"Stale write detected — recover or discard?"`. |

---

## §5 Edge cases

- **Topic too vague** (`/create grow`) — clarification before any work.
- **Topic too broad** (`/create become a billionaire`) — out-of-scope; suggest narrower phrasing.
- **Non-supported domain** (fitness, business) — decline politely.
- **Roadmap clobber** — see Step 9.
- **Concurrent runs** — see Step 9.
- **Non-English topics** — supported. Persona should be region-aware where the topic implies a market (e.g., "20→50 LPA" → Indian tech market).

---

## §6 Other modes (not yet released)

If invoked in `refresh` or `review` mode in this version, tell the user:

> "{Mode} mode lands in a future release — not yet implemented. For now, you can manually edit the markdown file, run `/claudemap-coach:update` to mark progress, or re-run `/claudemap-coach:create` with a fresh topic."

Do NOT attempt to half-implement these modes.

---

## §7 References

- `templates/roadmap.md.tmpl` — markdown skeleton.
- `templates/progress.json` — sidecar JSON schema.
- `references/trend-search-patterns.md` — WebSearch query templates.
- `references/model-pricing.md` — token cost lookup.
