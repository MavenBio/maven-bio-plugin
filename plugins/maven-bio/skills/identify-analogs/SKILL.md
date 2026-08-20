---
name: identify-analogs
description: "Find historical or competitive precedents that inform an asset, company, or market question, and explain why each is a fair comparison. Use for precedent reasoning, for example 'what is the closest analog to this launch', 'what should we price off'. Use benchmark-assets for a direct side-by-side comparison of named assets instead."
---

# Identify Analogs

This primitive finds reference cases that help frame expectations without forcing a final output format.

## Use When

- a workflow needs comparable launches or programs
- the user asks for historical analogs
- you need precedent cases to anchor sizing, positioning, or forecast logic

## Core Tools

- `match_entity`
- `search_entities`
- `research_entity`
- `fetch_related`
- `search_documents`
- `read_document`
- `get_recent_events` (when the analogy turns on how a comparable asset's milestones actually unfolded)

Use `match_entity` when you already know the specific entity you want to anchor analog selection around. Keep the raw entity name in `name` and put sponsor/company/disambiguating text in `context`. Use `search_entities` when you are discovering analog candidates by criteria.

## Output Contract

Return a structured analog set with:

- analog entity
- why it is comparable
- evidence-backed dimensions of similarity
- differences or boundary conditions
- evidence ratings and citations

## Optional Primitives

- `validate-target` (when analog selection depends on shared, genetically-validated targets)

## Quality Bar

- explain why each analog belongs in the set
- avoid superficial similarity without mechanism or market logic
- keep disanalogies visible
- treat analog selection as an analytic judgment supported by evidence, not as a fact
- timing-based analogies (LOE-driven generic entry, biosimilar windows, post-exclusivity uptake) require comparable regulatory milestones on both the anchor and each analog; source those milestones from FDA or SEC filings via `search_documents` plus `read_document` before drawing a timing inference, and drop the analogy rather than inferring a date neither filing supports
