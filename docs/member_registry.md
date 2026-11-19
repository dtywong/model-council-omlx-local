# Council member registry — template

[香港廣東話](member_registry_zhHK.md)

A council needs a registry: one entry per model you can call as a "seat," with enough
detail that convening a council is a lookup, not a re-decision every time. This is a
template showing the shape of that registry, not a specific person's live roster.

Each entry should record:

- **How to call it** — the actual invocation (an API call, a CLI command, an SDK client),
  kept runnable, not just described in prose.
- **Whether it participates in peer review** — some models are answering-only (no
  structured ranking output wired up yet, or deliberately excluded because their rankings
  add more noise than signal).
- **What it's a default seat for** — which task types this model has actually proven itself
  on, versus task types it's untested for. Don't extend a model's proven strength on one
  task type to another without evidence.
- **Known quirks** — rate limits, token-budget floors for reasoning models, encoding issues,
  anything that has bitten a real call before.

## Example entries (illustrative shape, not a real roster)

### claude-tier-a — a fast, capable general-purpose model
- Call: your agent framework's standard dispatch, requesting the mid-tier model.
- Reviews: yes.
- Default seat for: all task types. The dependable default answering seat.

### claude-tier-b — a stronger, heavier model in the same family
- Call: same dispatch mechanism, requesting the top-tier model.
- Reviews: yes.
- Default seat for: judgment-heavy or high-stakes questions. Skip for routine work where
  the faster tier already suffices — reserve the heavier model deliberately, don't default
  to it everywhere.

### open-model-a — a different model family entirely
- Call: whatever that provider's API/CLI wrapper looks like.
- Reviews: yes, if its output can be coerced into the same ranking format the others use;
  otherwise answering-only until that's built.
- Default seat for: all task types — a genuinely different "vote," not just another
  instance of the same underlying training approach.

### cloud-frontier-x / cloud-cheap-y — models reached via a shared cloud inference API
- These typically come in tiers: a frontier tier (worth including in a premium council) and
  a lower-cost tier (fine for a budget council, generally NOT worth putting next to a
  frontier/flagship model in the same council — see the dilution finding in
  [`pattern.md`](pattern.md)).
- Note model IDs somewhere versioned and re-checked periodically. Provider catalogs churn;
  an ID that's live today can go stale within months.

### search-grounded-seat — a model whose answer comes from live web search rather than
  parametric knowledge
- Reviews: no (it isn't a ranker, it's a research seat).
- Default seat for: questions that genuinely need current facts or citations. Skip for pure
  reasoning or writing tasks where it adds nothing.

## Default seat combinations, as a starting point

| Council type | Suggested seats | Chair |
|---|---|---|
| Premium (highest-stakes judgment) | one strong general model + 2 frontier-tier seats from a different lineage | the strong general model |
| Budget (cost-constrained, still real judgment) | 3 lower-cost seats from at least 2 distinct lineages | your fastest capable model |
| Anti-pattern — never | a strong model + 1-2 mid-tier seats "for extra opinions" | — scored below the strong model alone in practice; don't do this |

Tune this against your own evidence as you accumulate it (see
[`scorecard_schema.md`](scorecard_schema.md)) — these are reasonable starting points, not
fixed rules.
