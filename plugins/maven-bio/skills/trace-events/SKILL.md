---
name: trace-events
description: "Return the recent event stream (FDA actions, trial readouts, press releases) for a named drug, company, trial, or indication, with structured evidence per event. Use for recency questions, for example 'what happened with X recently', 'any news on this program'. This is a building block the workflow skills compose."
---

# Trace Events

This primitive is for timeline-style intelligence and recent developments.

## Use When

- the user asks what happened recently
- a workflow needs catalysts, recent news, or regulatory updates
- you need a recent-event layer around an entity or indication

## Core Tools

- `get_recent_events`
- `read_document`
- `search_documents`

## Output Contract

Return a structured event list with:

- date
- event type
- summary
- evidence rating
- citations
- unresolved questions

## Quality Bar

- use event feeds for discovery
- strengthen material claims with document reads when needed
- keep chronology explicit
- preserve ambiguity if event interpretation is uncertain
