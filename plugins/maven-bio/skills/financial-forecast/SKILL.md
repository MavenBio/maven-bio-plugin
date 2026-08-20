---
name: financial-forecast
description: "Assemble the assumptions behind a revenue or valuation forecast: population, penetration, pricing, timing, and competitive erosion, each with its evidence. Use when someone is building a model, for example 'what assumptions should I use for X', 'help me forecast peak sales'. Returns assumptions, not a finished spreadsheet or narrative. Use size-market when only sizing is wanted."
---

# Financial Forecast

This workflow is for forecast intelligence, not spreadsheet rendering.

## Primary Primitives

- `size-market`
- `identify-analogs`
- `enumerate-entities`
- `trace-events`
- `synthesize-evidence`
- `profile-entity`

## Optional Primitives

- `benchmark-assets`

## Output Contract

Return a forecast assumption package that can include:

- market assumptions
- analog references
- asset-specific development or competitive assumptions
- scenario caveats
- evidence ratings and citations for material assumptions
- unresolved forecast sensitivities

## Workflow Rules

- treat structured MCP outputs as the starting assumption stack, not the final forecast truth
- use document augmentation for pricing, analog, catalyst, and competitor assumptions that materially affect the package
- if host web fetch is blocked, use `search_documents` and `read_document` as the fallback path before declaring a gap
- when assumptions rest on independent evidence workstreams (population, pricing, competitive erosion, timing), run them in parallel with the `research-assistant` sub-agent, then reconcile the returned claim sets before stating the assumption set
