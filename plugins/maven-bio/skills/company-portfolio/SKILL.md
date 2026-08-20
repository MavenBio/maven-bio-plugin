---
name: company-portfolio
description: "Build a full intelligence package on one company: pipeline, recent events, capital position, and strategic posture. Use when someone names a company and wants depth, for example 'profile Vertex', 'what is going on at X', 'walk me through their pipeline'. Use profile-entity instead when only a single structured fact sheet is wanted."
---

# Company Portfolio

This workflow produces company-level intelligence without assuming a report or table deliverable.

## Primary Primitives

- `profile-entity`
- `enumerate-entities`
- `trace-events`
- `trace-financings`
- `synthesize-evidence`

## Optional Primitives

- `identify-analogs`
- `benchmark-assets`
- `synthesize-evidence` (when LOE timing, exclusivity calendar, or label-extension opportunities shape the strategic posture and each date needs a source)

## Output Contract

Return a company intelligence package that can include:

- company overview
- pipeline snapshot
- recent events
- capital posture (recent rounds, total raised in window, lead investors, financing trajectory, runway implication)
- public-market financial caveats when applicable (market cap, cash position, revenue scale)
- strategic themes
- discussion topics or diligence gaps when relevant

## Aggregation Shortcuts

For the capital-trajectory and pipeline-shape views of one company, prefer `aggregate_records` over manually aggregating `research_entity(..., aspects=["financings"])` rounds:

- Financing trajectory: `aggregate_records(entity_type="financing", query="rounds for {company} by year with total value")`
- Pipeline shape: `aggregate_records(entity_type="drug", query="active programs for {company} by phase and indication")`

## Workflow Rules

- treat company/entity profiling as the baseline fact pattern, not the final synthesis
- pair `aspects=["financials"]` with `aspects=["financings"]` on `research_entity` for the public and private capital pictures; for VCs, corporate venture arms, and strategic investors, also pull `aspects=["investments"]`
- treat the `related_previews.financings` and `related_previews.investments` teaser on the default company response as a routing prompt, not as evidence; opt into the full aspect when the funding picture matters to the analysis
- on every financing row, separate the confirmed-value slice from the unconfirmed slice; never aggregate `total_value_usd: null` rounds into a "total raised" headline without footnoting
- augment important company, pipeline, event, and round-level claims with `search_documents` and `read_document`; the structured row is metadata, not citable evidence
- for portfolios with approved assets, surface upcoming patent expiries, biosimilar entry windows, and label-extension opportunities from primary filings via `search_documents(source_types=["fda_filings", "sec_filings"])` and `read_document`; where no filing supports a date, record it as a gap rather than estimating it
- if host web fetch is blocked, prefer Maven-indexed document retrieval over repeated blocked fetch attempts
- when the company splits into independent evidence workstreams (pipeline, events, capital position, deals), run them in parallel with the `research-assistant` sub-agent, then reconcile the returned claim sets before synthesis

## BD and Diligence Posture

For BD, corp dev, and acquisition-target diligence, the company portfolio should surface a runway-implication slice in addition to the pipeline picture:

- the most recent round (date, type, value, lead investor) versus the company's burn signals from `aspects=["financials"]`
- the financing trajectory (accelerating, flat, or aging since last round)
- investor concentration and lead-investor signal
- the comp-set position relative to peers identified through `identify-analogs` or `enumerate-entities`

A capital-thin or aging-Series-B+ recipient with active mid-stage trials is a candidate signal. A well-funded recipient with a recent strategic investor is a different posture. Make the difference visible rather than collapsing both into "company overview."

For investment-team workflows oriented around capital momentum across a space rather than one company, route to `funding-landscape` instead of building one company portfolio at a time.
