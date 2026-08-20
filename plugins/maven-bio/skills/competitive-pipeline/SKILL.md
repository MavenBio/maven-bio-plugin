---
name: competitive-pipeline
description: "Map who is developing what for an indication, mechanism, or target, grouped by phase, company, or mechanism. Use for competitive questions, for example 'map the NSCLC landscape', 'who else is working on TL1A', 'what is the competition for X'. This is drug-program-centric: use indication-research when the question is about the disease itself, and funding-landscape when it is about capital flow."
---

# Competitive Pipeline

This workflow coordinates primitives to produce a reusable pipeline intelligence package.

The pipeline view is drug-program-centric (which assets are in development, at what phase, with what mechanism). For the capital-flow view of the same space (recent rounds, total raised, investor mix, deal cadence), route to `funding-landscape` instead. The two layers are independent and a complete competitive picture often needs both.

## Primary Primitives

- `enumerate-entities`
- `profile-entity`
- `synthesize-evidence`

## Optional Primitives

- `trace-events`
- `benchmark-assets`
- `synthesize-evidence` -- when the pipeline question includes per-row evaluation criteria beyond catalog metadata (e.g., "which of these have an oral formulation", "which target KRAS G12C specifically"); narrow the set before evaluating rather than thinning the evidence per row
- `validate-target` -- when the pipeline view ranks programs by target genetic validation or specificity

## Output Contract

Return a structured package that can include:

- scoped asset universe
- core confirmed asset set
- watchlist / uncertain asset set
- per-asset evidence-backed profiles
- verification gaps and rows needing follow-up
- key competitive dimensions
- recent catalysts or notable events
- gaps, exclusions, and scope caveats

For pipeline tables or asset lists, use two tiers:

- **Core confirmed**: assets with indication-relevant trial identifiers in the structured payload, or assets confirmed in-session with `read_document`, `research_entity`, or another document/read path.
- **Watchlist / uncertain**: assets without indication-relevant trial identifiers, assets supported only by discovery/search snippets, ambiguous sponsor/name/status matches, conflicting evidence, or assets that need follow-up before promotion.

## Aggregation Shortcuts

For the shape of the pipeline (counts by phase, by mechanism, by sponsor) rather than per-asset detail, use `aggregate_records(entity_type="drug", ...)`. This complements `research_landscape` (which returns the asset list) by giving a chart-ready distribution in a single call.

- Phase distribution: `query="active drugs in {indication} grouped by phase"`
- Mechanism crowding: `query="active drugs in {indication} grouped by mechanism, top 25"`
- Sponsor concentration: `query="active drugs in {indication} grouped by company, top 25"` (note: `companies` is M2M-exploded; co-developed assets count once per owning company)

`aggregate_records` is for the count layer, not the evidence layer. Any quoted statistic still needs a document or trial identifier behind it before it goes into the core confirmed tier.

## Core Research Pattern

This workflow should follow a three-step pattern:

1. **Enumerate** the baseline pipeline universe with structured MCP tools. For the aggregate shape (phase distribution, mechanism crowding, sponsor concentration), use `aggregate_records` to get the picture before enumerating individual assets.
2. **Augment** that baseline with `search_documents`, `read_document`, and event/document checks
3. **Reconcile** the final asset set and key claims before returning the package

`research_landscape` and related structured tools are the right starting point, but they are not the full truth source for a master pipeline analysis. Use document search to surface:

- assets missing from the structured layer
- recent status changes or new disclosures
- naming, ownership, or mechanism discrepancies
- stronger support for the highest-impact competitive claims

## Workflow Rules

- treat scoping as a first-class step
- keep inclusion logic explicit
- treat structured search as the baseline universe, not the final truth
- require a document augmentation pass before finalizing the package
- do not finalize a pipeline after only `research_landscape`, `search_entities`, `fetch_related`, or entity profiles
- before final output, run a scope-reconciliation gate:
  - compare the returned row count to the expected order of magnitude implied by the prompt or prior context
  - check that major mechanism/modality classes and phase/status buckets requested by the user are represented
  - check known anchor assets from the prompt, retrieved documents, or domain context
  - decide whether missing anchors are true exclusions, unresolved gaps, or a signal to broaden the search
- split pipeline outputs into core confirmed rows and watchlist / uncertain rows when row-level confidence varies
- do not promote a structured-search row without an indication-relevant NCT or trial identifier into the core confirmed tier unless you confirmed it in-session with `read_document`, `research_entity`, or another read/fetch path
- confirm key per-asset facts from `search_entities` and `research_landscape` with `search_documents` plus `read_document` or `research_entity` before treating them as evidence-backed claims
- reconcile contradictions between structured outputs and document findings
- use evidence synthesis for the claims that matter most
- avoid assuming a final report, table, or chart format
- group programs by trial phase AND approval status. An approved drug is not Phase 4; for any asset that may already be approved in a covered region, confirm against the catalog phase (`Approved`/`Filed`) and, where the distinction carries the answer, against an FDA filing via `search_documents` plus `read_document`
- once the comparator set is fixed, gather per-program evidence in parallel with the `research-assistant` sub-agent, one workstream per program, then reconcile the returned claim sets before building the table

## Fallback Rules

- If you are resolving one named asset, company, or indication, use `match_entity`; reserve `search_entities` for broader criteria-based discovery.
- When calling `match_entity`, keep the raw entity name in `name` and put sponsor/company/disambiguating text in `context`.
- When calling `research_landscape`, keep the raw indication name in `indication` and put extra disambiguating text in `context`.
- If `research_landscape` or related structured enumeration looks too small, run a secondary `search_entities` pass and broad `search_documents` augmentation before finalizing.
- If structured enumeration looks large but noisy, keep the broad set as discovery scaffolding, then use document evidence and explicit exclusion rules to separate core confirmed rows from watchlist rows.
- If a named asset fails to resolve, retry with the raw entity name and move sponsor/company text into the optional context hint.
- If host web fetch is blocked, treat that as expected Cowork behavior and pivot to `search_documents` plus `read_document`.
