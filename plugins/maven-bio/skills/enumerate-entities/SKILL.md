---
name: enumerate-entities
description: "Build the scoped list of drugs, companies, trials, mechanisms, or targets matching a set of criteria, without analyzing each one. Use when the job is to define the right set first, for example 'list every Phase 3 KRAS G12C program'. This is a building block: use competitive-pipeline when the landscape should be analyzed rather than listed."
---

# Enumerate Entities

This primitive is for **scope definition**, not final deliverable formatting.

## Use When

- the user wants the universe of entities in a space
- you need the candidate set before benchmarking or profiling
- the first problem is recall and scoping, not narrative synthesis

## Core Tools

- `match_entity`
- `search_entities`
- `research_landscape`
- `fetch_related`
- `get_financings` for financing-criterion enumeration (e.g., recipients of recent Series B+ rounds in a space, or rounds led by a specific investor set)

## Output Contract

Return a structured entity list with:

- entity id
- entity name
- entity type
- why it is in scope
- evidence status for the inclusion decision
- gaps or ambiguity notes

This primitive is allowed to return `citation_status: unresolved` when the output is still discovery scaffolding rather than document-backed evidence.

## When Enumeration Is Not Enough

Enumeration answers "which entities are in scope". It does not answer questions that
require per-row evaluation against custom criteria rather than catalog metadata:

- the user wants to filter the enumerated set by conditions that require LLM judgment (e.g., "targets KRAS G12C specifically", "has an oral formulation")
- the user wants enrichment dimensions extracted per entity with evidence (e.g., "lead indication", "differentiating feature")
- the evaluation criteria are custom to the question, not fixed catalog fields like phase or mechanism

`enumerate-entities` defines the candidate universe; `synthesize-evidence` evaluates it. When the universe is larger than the evaluation budget, narrow the enumeration criteria rather than reducing the evidence gathered per entity.

## Baseline, Not Completion

Enumeration defines the **baseline universe**. It does not, by itself, complete a master analysis.

For broad analyses such as landscapes, pipelines, benchmark sets, or strategic maps:

- use this primitive to establish the candidate set
- then run a document augmentation pass with `search_documents`, `read_document`, and relevant event checks
- reconcile newly surfaced entities, recent changes, or contradictory evidence before treating the set as final
- run a scope-reconciliation gate before downstream finalization: compare the candidate count, class coverage, phase/status coverage, and known anchor entities against the requested scope
- if the candidate set is suspiciously sparse, too broad, or missing expected anchors, treat that as a research gap and broaden/narrow the search before finalizing

If the entity list is still only discovery scaffolding, keep that explicit in the output.

## Fallback Rules

- If a resolver-backed tool fails on a composite name, retry with the raw entity name and move sponsor/company text into the optional context hint.
- Use `match_entity` when the task starts from one known entity name and you need the canonical Maven entity.
- When calling `match_entity`, keep the raw entity name in `name` and put sponsor/company/disambiguating text in `context`.
- When calling `research_landscape`, keep the raw indication name in `indication` and put extra disambiguating text in `context`.
- Use `search_entities` when the task is to discover or enumerate entities by criteria, filters, or market description.
- For financing-criterion enumeration (recipients of rounds, investor-led sets, capital-flow scoping in a space), use `get_financings` directly when the question is structured (recipient / investor / type / value / date), or `search_entities("financing", ...)` when the question is a natural-language phrasing. The recipient-side ontology axes (indication, modality, mechanism, target) are reachable through `search_entities("financing", ...)` even when not exposed as typed `get_financings` params.
- If the baseline set looks suspiciously sparse, follow with `search_entities` and a document augmentation pass.
- If the baseline set looks noisy or over-broad, preserve a watchlist/exclusions tier rather than collapsing every candidate into the core set.
- If document search surfaces new candidates not seen in the structured layer, preserve them explicitly for reconciliation.

## Aggregation Shortcut

Once a universe is scoped, `aggregate_records(entity_type=X, query="...grouped by Y")` answers "how many in each bucket" without paginating the full enumeration. Use it as a sanity check on the size and shape of the enumerated set before downstream skills consume it.

## Quality Bar

- use explicit scoping criteria
- separate in-scope, borderline, and excluded entities when useful
- do not present enumeration as proof of a downstream factual claim
- do not treat structured enumeration as the final truth for a master analysis without a document augmentation pass
- do not treat an entity set as complete until the scope-reconciliation gate has been run or the remaining uncertainty is stated explicitly
- note ambiguity when the entity set may be incomplete
