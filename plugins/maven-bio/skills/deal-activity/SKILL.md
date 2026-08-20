---
name: deal-activity
description: "Trace BD deals (licensing, M&A, R&D collaborations, joint ventures) for a company, asset, indication, or deal type, and return structured deal-level evidence with provenance. Use for transaction questions, for example 'who has licensed X', 'what deals have been done in obesity'."
---

# Deal Activity

This primitive turns a business-development question into a structured deal set with explicit value disclosure status, party roles, and document provenance.

## Use When

- the question is fundamentally about deals or transactions, not financing rounds, drug programs, or trials
- a workflow needs cross-company BD activity for a company, an asset, an indication, or a deal type
- you need deal-level evidence (date, type, parties and their roles, value, supporting documents) before reasoning about strategy, valuation benchmarks, or BD momentum

## Core Tools

- `get_deals` for structured cross-company filters (party `company`/`company_id` plus `role`, `deal_type`, USD value range, date window, sort by `total_deal_value_usd`)
- `search_entities("deal", ...)` for natural-language deal discovery and fuzzy phrasings the structured params cannot express
- `research_entity(name, "company", aspects=["deals"])` for the deals a named company is a party to, with a per-row role
- `get_recent_events(entity_name=...)` for the recency-ordered news feed when the question is "what was announced lately"
- `read_document` against the deal's `document_ids` to back any material claim

## Tool Choice

- if the agent already knows the named company, use `research_entity(..., aspects=["deals"])` -- one credit, one shape, no resolution overhead
- if the question is structured ("licensing deals in oncology since 2024 over $500M", "every deal where Pfizer is the out-licensor", "M&A over $1B"), use `get_deals` with typed filters and `role`
- if the question is fuzzy or compositional ("immuno-oncology dealmaking momentum", "recent platform tie-ups"), use `search_entities("deal", ...)` and let the natural-language path translate it
- for "what's new this week" feed-style questions, use `get_recent_events` rather than `get_deals`; the latter is structured search, the former is recency-ordered

## Output Contract

Return a structured deal list with:

- `deal_id` (the prefixed `deal_` identifier)
- `event_date`
- `deal_type` (canonical: Licensing, R&D collaboration, Manufacturing collaboration, Commercialization agreement, Joint venture, M&A - company, M&A - product/asset, Spinoff, Platform/technology access, Service agreement, Other) and `deal_subtype` where present
- parties as `{id, name, role}` (roles: Licensor, Licensee, Collaborator, Acquirer, Acquiree, Investor, Investee, Manufacturer, Distributor, Sponsor, Partner, Seller, Service Provider, Customer)
- `total_deal_value_usd` and `value_status` (`"confirmed"` or `"unconfirmed"`); near-term and milestone payment fields follow the same value-status contract
- linked drugs, indications, and trials as `{id, name}` pairs
- supporting `document_ids` for chaining into `read_document`
- evidence rating per material claim
- explicit gaps for unconfirmed values, missing counterparties, or contradictory sources

## Aggregation Shortcut

If the question is "totals, distributions, or rankings across a deal slice," prefer `aggregate_records(entity_type="deal", ...)` and use `get_deals` to fetch deal-level evidence for the claims you cite. Party fields are M2M-exploded (a deal with three parties counts in all three groups); separate the confirmed-value denominator rather than treating an unconfirmed value as zero.

## Quality Bar

- never present `total_deal_value_usd: null` as "the deal was undisclosed" -- the correct phrasing is "Maven's data does not have the value." The deal may have a publicly disclosed amount Maven has not captured. Verify externally if the value is critical.
- when reporting aggregate totals (sum of deal value, average upfront), separate the confirmed-value slice from the unconfirmed slice
- treat the canonical deal types and roles as closed enums; if a caller-supplied alias resolves to a canonical value ("M&A" -> "M&A - company", "buyer" -> "Acquirer", "out-licensor" -> "Licensor"), surface the resolution explicitly so downstream readers see the normalization
- for deal-level claims (upfront vs milestones vs royalties, biobucks headline, territory scope), require document evidence from `read_document`; the structured row alone is metadata, not citable evidence
- distinguish a deal's headline `total_deal_value_usd` (often a biobucks potential) from realized economics; do not conflate total potential with upfront
- when sorting by deal size, use `sort_by="total_deal_value_usd"` and note that unconfirmed-value deals are ordered last

## Fallback Rules

- if a party name does not resolve, the response surfaces ranked candidates under `unresolved_filters` keyed per input. Retry with a candidate name in one round-trip rather than guessing; if every party name is unresolved the call errors with `entity_not_found`
- if a `deal_type` or `role` value does not match the canonical enum, the response either auto-corrects (case-fold or alias) and surfaces the correction, or rejects with the canonical list. Use the canonical list, do not invent new types or roles
- if `get_deals` returns zero rows for a structured query, retry with `search_entities("deal", "<the same intent in natural language>")` before concluding no deals exist
- if host web fetch is blocked, prefer `search_documents` plus `read_document` against the deal's `document_ids` rather than retrying blocked external fetches
- if a deal question remains blocked after the above, state the gap explicitly in the output rather than substituting a weaker proxy answer
