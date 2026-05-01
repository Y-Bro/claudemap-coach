# Model Pricing

Reference table for converting token counts into approximate cost in the `## Run Stats` block. Prices are USD per **1 million tokens**.

> ⚠️ **Verify before relying on this.** Anthropic publishes the canonical price list. The values below are best-effort; confirm against https://www.anthropic.com/pricing and update this file when prices change.

## Current entries

| Model ID | Input | Output | Cache write (5m TTL) | Cache read |
|---|---|---|---|---|
| `claude-opus-4-7` | $15.00 | $75.00 | $18.75 | $1.50 |
| `claude-opus-4-7-1m` | $30.00 | $150.00 | $37.50 | $3.00 |
| `claude-sonnet-4-6` | $3.00 | $15.00 | $3.75 | $0.30 |
| `claude-haiku-4-5` | $0.80 | $4.00 | $1.00 | $0.08 |
| `claude-haiku-4-5-20251001` | $0.80 | $4.00 | $1.00 | $0.08 |

**Last verified:** 2026-05 (placeholder — verify against the official pricing page before any release).

## Cost calculation

```
total_cost_usd =
    (input_tokens          / 1_000_000) * input_price
  + (output_tokens         / 1_000_000) * output_price
  + (cache_creation_tokens / 1_000_000) * cache_write_price
  + (cache_read_tokens     / 1_000_000) * cache_read_price
```

The `message.usage` object on each assistant message in the session transcript exposes:
- `input_tokens` — fresh input tokens this turn (not cached)
- `output_tokens` — generated tokens
- `cache_creation_input_tokens` — input tokens written to the cache
- `cache_read_input_tokens` — input tokens served from the cache

## Unknown model

If the active model is not in the table above:

1. Output `Approx cost: unknown (model '<id>' not in pricing table)`.
2. Tell the user to update `skills/roadmap-coach/references/model-pricing.md` to add the model.
3. Do not invent a price.

## Updating

When Anthropic publishes new pricing or new models:
1. Update the rows above.
2. Bump the **Last verified** date at the top.
3. Note the change in `CHANGELOG.md`.
