---
name: asset-profile
description: "Build a full intelligence packet on one named drug or asset: development status, trial evidence, competitive position, catalysts, and risks. Use when someone names a single asset and wants depth, for example 'tell me everything about X', 'profile this drug', 'is X differentiated'. Use profile-entity instead when only a single structured fact sheet is wanted."
---

# Asset Profile

This workflow produces a structured profile for one asset without assuming a final memo or deck format.

## Primary Primitives

- `profile-entity`
- `benchmark-assets`
- `trace-events`
- `synthesize-evidence`

## Optional Primitives

- `identify-analogs`
- `size-market`
- `synthesize-evidence` (when the asset is approved or near-approval and designation, label, or exclusivity claims need to be tied back to specific filings)
- `validate-target` (when the asset's thesis depends on its target's genetic validation)

## Output Contract

Return a structured asset intelligence package that can include:

- asset overview
- development status
- supporting trial or document evidence
- competitive framing
- recent catalysts
- key risks and unresolved questions

## Workflow Rules

- treat entity profiling as the baseline fact pattern, not the final synthesis
- when calling `research_entity`, keep the raw asset name in `name` and put sponsor/company/disambiguating text in `context`
- use document augmentation to strengthen trial, catalyst, forecast, and competitive claims
- if host web fetch is blocked, prefer `search_documents` and `read_document` over repeated blocked fetch attempts
- if a composite asset name fails to resolve, retry with the raw asset name and move sponsor/company details into the optional context hint
- if the asset is approved in any region, include a regulatory timeline (designations, approval dates, label revisions) built from `search_documents` with `source_types=["fda_filings"]` and `read_document`; cite the filing and its date for each milestone, and state exclusivity or LOE timing as a gap when no filing supports it
- when the asset splits into independent evidence workstreams (trials, catalysts, competitors, target validation), run them in parallel with the `research-assistant` sub-agent, then reconcile the returned claim sets before synthesis
