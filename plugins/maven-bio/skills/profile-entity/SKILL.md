---
name: profile-entity
description: "Return one structured, evidence-backed fact sheet for a single known drug, company, trial, mechanism, or target. This is a building block the workflow skills compose. Use it directly only when someone wants the raw fact pattern for one entity; for a full asset picture use asset-profile, and for a full company picture use company-portfolio."
---

# Profile Entity

This primitive turns one known entity into a structured fact pattern with explicit provenance.

## Use When

- the user wants depth on a specific named entity
- a workflow needs entity-level facts before synthesis
- you need mechanism, trial, event, document, or per-indication development detail tied to one entity

## Core Tools

- `research_entity`
- `search_documents`
- `read_document`
- `get_recent_events`
- `search_documents` with `source_types=["fda_filings"]` (drug entities, when an approval, designation, or label snapshot matters)

## Output Contract

Return structured findings such as:

- entity summary
- key facts
- evidence-backed claims
- important unknowns

Each claim should include:

- `claim`
- `evidence_rating`
- `citations`
- optional `notes`

## Quality Bar

- use `research_entity` for discovery and structure
- request `aspects=["indication_phases"]` on `research_entity` when you want the asset's per-indication phase/status footprint
- for company entities, request `aspects=["financings"]` for rounds the company received and `aspects=["investments"]` for rounds the company invested in (useful for VCs, corporate venture arms, and strategic investors). Both are opt-in
- the default company response includes a lightweight `related_previews.financings` and `related_previews.investments` teaser (count plus a few sample rounds). Treat these as prompts for whether to opt into the full per-round detail rather than as evidence in their own right
- on every financing row, `value_status` disambiguates `total_value_usd`. `"confirmed"` means Maven has a USD value. `"unconfirmed"` means Maven's data does not have the amount, not that the round was undisclosed in the world. Verify externally if the value is critical
- distinguish `aspects=["financials"]` (public-market financial statements: revenue, EBITDA, market cap, cash) from `aspects=["financings"]` (private/round-level capital activity). Both can apply to the same company; pair them when reasoning about runway or capital posture
- treat `related_previews.documents` and `related_previews.events` as prompts for what to inspect next when you requested a narrower aspect
- use `search_documents` and `read_document` when a claim needs evidence
- for approved drugs, build the approval snapshot from primary filings via `search_documents(source_types=["fda_filings"])` plus `read_document`, and cite the document and its date for any label-sourced claim; report exclusivity or LOE timing as a gap when no filing supports it
- keep the raw entity name in the main tool field and move sponsor/company text into the optional context hint
- do not treat entity metadata alone as direct evidence
- if host web fetch is blocked, use `search_documents` and `read_document` instead of treating the block as the end of the research path
- keep missing or weakly supported facts visible as gaps
