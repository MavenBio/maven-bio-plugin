---
name: size-market
description: "Estimate patient population and market size with explicit assumptions, evidence strength, and remaining uncertainty. Use when sizing is the question, for example 'how big is the market for X', 'how many patients are eligible'. Use financial-forecast when revenue projection assumptions are wanted, and indication-research for the broader disease picture."
---

# Size Market

This primitive produces structured population and market-sizing assumptions, not a finished financial model.

## Use When

- the user asks for prevalence, incidence, addressable population, or market size
- a workflow needs sizing assumptions before forecast or prioritization
- you need an explicit epidemiology funnel with evidence discipline

## Core Tools

- `match_entity`
- `search_documents`
- `read_document`
- `search_entities`
- `research_entity`

Use `match_entity` when the market-sizing task starts from one known asset, company, or indication. Keep the raw entity name in `name` and put sponsor/company/disambiguating text in `context`. Use `search_entities` when you need to discover a broader entity set by criteria.

## Output Contract

Return a sizing object with:

- population layers or funnel steps
- assumption values
- evidence ratings
- citations
- caveats

## Quality Bar

- show the decomposition rather than only the final number
- cite assumptions at the layer where they enter the logic
- label extrapolations or weak steps honestly
- preserve uncertainty instead of false precision
