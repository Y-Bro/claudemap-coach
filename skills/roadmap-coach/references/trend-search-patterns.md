# Trend Search Patterns

WebSearch query templates the `roadmap-coach` skill (and the `roadmap-specialist` agent) use to ground roadmap content in current reality. `{YYYY}` should be replaced with the current four-digit year at runtime.

## Categories

### Industry trends (run during create + refresh)
- `{topic} job market trends {YYYY}`
- `{target_role} skills in demand {YYYY}`
- `skills gap {target_role} {YYYY}`
- `{topic} hiring requirements {YYYY} site:linkedin.com OR site:levels.fyi`

### Skill stack currency
- `{tech_or_skill} vs {alternative} {YYYY}`
- `deprecated {tech} {YYYY}` — flags stale content
- `{topic} most popular frameworks {YYYY}`

### Reputable resources (with link verification)
- `best {topic} courses {YYYY} reddit OR hackernews`
- `{specific_skill} course recommendations {YYYY}`
- `{topic} books recommended by senior engineers {YYYY}`
- `{topic} certification value {YYYY}`

### Compensation grounding (only for salary-target roadmaps)
- `{role} salary {location} {YYYY} site:levels.fyi OR site:glassdoor.com`
- `{role} compensation bands {company_tier} {YYYY}`

### Refresh-specific (only in refresh mode)
- `what's new in {topic} since {lastRefreshed_date}`
- `{topic} announcements {YYYY-MM}`

## Domain preferences

When the WebSearch tool supports `allowed_domains` / `blocked_domains`, prefer:

### Salary queries — `allowed_domains`
- `levels.fyi`
- `glassdoor.com`
- `payscale.com`

### Technical resources — `allowed_domains`
- `*.edu`
- `github.com`
- `oreilly.com`
- `manning.com`
- `coursera.org`
- `deeplearning.ai`
- `fast.ai`

### Block (low signal-to-noise)
- `medium.com/@*` (individual blogs vary wildly in quality; allow `medium.com/<publication>` if cited by multiple sources)
- `*.tutorialspoint.com`
- `*.javatpoint.com`
- Listicle aggregators ("top 10 ... in 2026" SEO farms)

## Orchestration rules

- **Pick 4–6 categories per run**, weighted to the topic shape:
  - Career transition → industry trends + skill stack currency + reputable resources
  - Salary target → all of the above + compensation grounding
  - Skill mastery → skill stack currency + reputable resources (heavy)
  - Certification → industry trends (`certification value`) + reputable resources (skip salary)
- **Batch as parallel WebSearch calls in a single message.** Do not run them sequentially.
- **Hard cap: 8 calls per `create` run** (the specialist gets a separate 5-call budget).
- **Save signals to the sidecar** (`trendSignals` field) so future `refresh` runs can diff against the baseline.

## Output extraction

From each batch of results, extract:

1. **Top in-demand skills** — frequency-rank skills mentioned across results.
2. **Recommended resources** — dedupe by URL; prefer reputable domains; carry the source URL.
3. **Salary anchors** — numeric ranges with source URL.
4. **Deprecated/declining items** — anything explicitly called out as "no longer recommended" or "fading."

## Resource link discipline

Every resource that lands in the final roadmap MUST:
1. Have a real URL pulled from a WebSearch result.
2. Be tagged with `verified-{YYYY-MM}` so future `refresh` runs know when to re-check.
3. Pass the specialist's verification pass (in-existence + reputable for the persona).

If a resource fails any of the above, omit it. **Do not invent URLs.**
