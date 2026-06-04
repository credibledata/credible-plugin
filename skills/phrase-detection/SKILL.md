---
name: phrase-detection
description: How to construct search targets for the get_context tool. Covers target-type classification and non-obvious decomposition patterns. Read the tool description for field definitions and the end-to-end…
---
# Search Target Construction for `get_context`

The `get_context` tool description defines each field and the two-phase workflow (source discovery → per-source entity drill-down). This skill focuses on the parts you won't get right by default: classifying concepts into target types and splitting ambiguous phrases.

**Scope of this skill:** the patterns below mostly apply to **phase 2 (entity drill-down)** — building `dimension` / `measure` / `view` / `dimensional_value` targets for a call scoped to a single source. Phase 1 (source discovery) is simpler: one or a few `source` targets describing the data domain, or a single `source` target with null `search_text` for listing. See the tool description for phase-1 guidance.

## Authoring `search_text` for `source` targets (phase 1)

Aim for **3-8 words that name the entity and its business process**. Don't include filter values, time ranges, or aggregations — those belong in phase-2 targets.

| Too vague | Over-specific (phase-2-ish) | Good |
|---|---|---|
| `"orders"` | `"total order revenue by customer last year"` | `"customer order history and line items"` |
| `"customer data"` | `"premium subscribers who churned in NYC"` | `"subscriber accounts and churn"` |
| `"metrics"` | `"monthly revenue variance by account"` | `"sales pipeline and revenue forecasts"` |

Heuristics:

1. **Translate, don't echo.** "How did sales go last month?" → `"sales order revenue"`, not `"sales last month"`.
2. **Differentiate by data shape or business process, not by the user's industry, brand, or product category.** Prefer `"order fulfillment and shipping"` over `"ecommerce data"` when multiple commerce-ish packages exist. Do NOT add words like `"eyewear"`, `"subscription box"`, `"Acme Corp"`, or the user's specific vertical/brand — source summaries describe data structure, not the customer's vertical, so those words add noise and can hurt matching.
3. **Retry with alternative phrasings** if the right source isn't in the results before concluding it's missing.

## Authoring `search_text` for entity targets (phase 2)

Write `search_text` as a brief semantic **description** of what you're looking for — not an echo of the user's word. For `dimensional_value` targets only, `search_text` is the **exact string to match**. This applies even when you already know the entity name from a prior result: still describe it, don't just repeat the name.

One target per concept is enough — the tool handles phrasing variants internally. Don't issue multiple `dimensional_value` targets for the same value just to try different wordings, and don't pile up dimension targets that point at the same field. Use multiple targets only when they describe genuinely distinct concepts (see "Non-obvious decomposition patterns" below).

## Target-type decision guide

- **`dimension`** — categorical attribute to group, filter, or join on. Also used for time and numeric fields.
  - "region" → `"the geographic region"`
- **`measure`** — aggregation metric (count, sum, average, rate).
  - "total revenue" → `"the total revenue or sales amount"`
- **`view`** — pre-built analysis. Include one whenever the question sounds like a canned report (summary, breakdown, top-N, trend).
  - "sales summary" → `"a summary of sales metrics"`
- **`dimensional_value`** — exact categorical value that appears as a literal string in the data. Only include this target when you have not yet pinned a dimension scope, OR when a prior `get_context` result shows the target dimension has `values_indexed: true`. Dimensions without `values_indexed: true` cannot be matched via dimensional value search — query their distinct values with `execute_query` instead.
  - "premium" → `"premium"`
- **`source`** — data domain. Used during source discovery, not drill-down (see tool description).

## Non-obvious decomposition patterns

These are the rules you won't apply correctly by default:

1. **Adjective + noun → split.** "active users" → three targets: a dimension for the attribute (`"the status of the user account"`), a dimensional value for the modifier (`"active"`), and a dimension for the noun (`"the user or account holder"`).
2. **Ambiguous concept → cover both types.** "rating", "duration", etc. could be either a dimension or a measure — create one target of each type.
3. **Time references are dimensions, never values.** "last year" → a dimension target for the relevant date field (`"the date the event occurred"`).
4. **Numeric ranges are dimensions, never values.** "aged 50", "revenue over $1M" → dimension targets. Numbers aren't matched as strings.
5. **Categorical strings that look numeric ARE values.** "18-30", "<5 days", "tier 2" — these appear as literal strings in the data → `dimensional_value`.
6. **"Top N" without a named measure → add a ranking measure.** "top 6 products" → a measure for the ranking concept (`"the performance metric for a product"`) plus a dimension for the entity. If the measure is explicit ("top products by total sales"), use it directly and skip the generic ranking measure.
7. **Multiple values for one concept → one target per value**, plus a single dimension target for the parent field.

## Worked example (drill-down call)

**User:** "Customer churn in NYC over the last year for premium and basic subscribers"

After source discovery surfaces a `subscriptions` source, the drill-down targets for the call scoped to it (`scopes` set to that source) are:

| target_type | search_text |
|---|---|
| `measure` | `"the rate at which customers leave the service"` |
| `dimension` | `"the city where the subscriber lives"` |
| `dimensional_value` | `"New York City"` |
| `dimension` | `"the date the subscription was canceled"` |
| `dimension` | `"the tier of the subscription"` |
| `dimensional_value` | `"premium"` |
| `dimensional_value` | `"basic"` |
| `view` | `"subscriber churn or retention analysis"` |

Key moves: time ("last year") becomes a dimension on the cancellation date, not a value; "premium/basic subscribers" splits into a tier dimension plus two dimensional values; one `view` target is included to surface any canned churn analysis.
