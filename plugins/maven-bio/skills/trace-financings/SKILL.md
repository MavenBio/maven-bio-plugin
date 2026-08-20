---
name: trace-financings
description: "Return the round-by-round financing record (amounts, dates, investors, provenance) for a named company, or filtered by indication, mechanism, target, or investor. Use when someone wants the actual rounds, for example 'list the funding rounds for X', 'who invested in their Series B'. Use funding-landscape when they want a market-level capital picture rather than a round list."
---

# Trace Financings

This primitive turns a funding question into a structured round set with explicit value disclosure status, investor mix, and document provenance.

## Use When

- the question is fundamentally about funding rounds, not drug programs or trials
- a workflow needs capital activity for a company, an indication, a mechanism, a target, or an investor
- you need round-level evidence (date, type, value, recipient, investors, supporting documents) before reasoning about runway, momentum, or investor signal

## Core Tools

- `get_financings` for structured cross-company filters (recipient, investor, financing_type, value range, date window, current-only, sort by total_value)
- `search_entities("financing", ...)` for natural-language financing discovery and fuzzy phrasings the structured params cannot express
- `research_entity(name, "company", aspects=["financings"])` for rounds where a named company was the recipient
- `research_entity(name, "company", aspects=["investments"])` for rounds where a named company was an investor (VCs, corporate venture arms, strategic investors)
- `get_financings` with a recent date window (newest first) for the recency feed of newly announced rounds -- financing recency is not in `get_recent_events`
- `read_document` against the round's `document_ids` to back any material claim

## Tool Choice

- if the agent already knows the named company, use `research_entity` aspects -- one credit, one shape, no resolution overhead
- if the question is structured ("Series B+ rounds in NSCLC since 2024", "rounds led by ARCH or Founders Fund", "rounds over $100M"), use `get_financings` with typed filters
- if the question is fuzzy or compositional ("late-stage immuno-oncology momentum", "biotech IPOs this year"), use `search_entities("financing", ...)` and let the natural-language path translate it
- for "what's new this week" feed-style questions, use `get_financings` with a recent date window; financing recency lives in `get_financings`, not `get_recent_events` (which covers only fda/trial/press)

## Output Contract

Return a structured round list with:

- `round_id` (the prefixed `fin_` identifier)
- `financing_date`
- `financing_type` (canonical: Pre-Seed, Seed, Series A, Series B, Series C, Series D+, IPO, Post-IPO Equity, Debt Financing, Grant, Strategic Investment, Acquisition, Other Financing)
- recipient companies as `{id, name}` pairs
- investor companies as `{id, name}` pairs
- `total_value_usd` and `value_status` (`"confirmed"` or `"unconfirmed"`)
- supporting `document_ids` for chaining into `read_document`
- evidence rating per material claim
- explicit gaps for unconfirmed rounds, missing investors, or contradictory sources

## Aggregation Shortcut

If the question is "totals, distributions, or rankings across a financing slice," prefer `aggregate_records(entity_type="financing", ...)` and use `get_financings` to fetch round-level evidence for the claims you cite. `median`/`avg` aggregations on `total_value` already separate the confirmed-value denominator via `_input_count`.

## Quality Bar

- never present `total_value_usd: null` as "the round was undisclosed" -- the correct phrasing is "Maven's data does not have the value." The round may have a publicly disclosed amount Maven has not captured. Verify externally if the value is critical.
- when reporting aggregate totals (sum raised, average round size), separate the confirmed-value slice from the unconfirmed slice rather than treating null as zero
- treat the canonical financing types as a closed enum; if a caller-supplied type alias resolves to a canonical value, surface the resolution explicitly so downstream readers see the normalization
- for round-level claims (lead investor, oversubscription, valuation, post-money), require document evidence from `read_document`; the structured row alone is metadata, not citable evidence
- the investor list may include thin-profile stub companies for previously-unknown investors -- these resolve via `research_entity` but with limited data; preserve the asymmetry rather than smoothing it over
- distinguish `get_financings` (private/round-level capital activity) from `get_financials` (public-market financial statements: revenue, EBITDA, market cap, cash). Both can apply to the same company; do not conflate them
- when sorting by deal size, use `sort_by="total_value"` and note that unconfirmed rounds are ordered last

## Fallback Rules

- if a recipient or investor name does not resolve, the response surfaces ranked candidates under `error.alternatives` keyed per input. Retry with a candidate name in one round-trip rather than guessing
- if a `financing_type` value does not match the canonical enum, the response either auto-corrects (case-fold or alias) and surfaces the correction, or rejects with the canonical list under `error.alternatives`. Use the canonical list, do not invent new types
- if `get_financings` returns zero rows for a structured query, retry with `search_entities("financing", "<the same intent in natural language>")` before concluding no rounds exist
- for indication / mechanism / target ontology axes, the recipient-side filters live on `search_entities("financing", ...)`; the structured `get_financings` surface does not expose them as typed params
- if host web fetch is blocked, prefer `search_documents` plus `read_document` against the round's `document_ids` rather than retrying blocked external fetches
- if a financing question remains blocked after the above, state the gap explicitly in the output rather than substituting a weaker proxy answer
