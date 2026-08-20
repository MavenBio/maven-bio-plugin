---
name: indication-research
description: "Build an evidence-backed picture of a disease: clinical context, epidemiology, unmet need, standard of care, and how the pipeline addresses it. Use for disease-first questions, for example 'tell me about ulcerative colitis', 'what is the unmet need in NASH'. Use competitive-pipeline when the question is really about competing drug programs, and size-market when only sizing is wanted."
---

# Indication Research

This workflow gathers the structured research needed to understand an indication without forcing a final document format.

## Primary Primitives

- `size-market`
- `enumerate-entities`
- `synthesize-evidence`

## Optional Primitives

- `trace-events`
- `profile-entity`
- `benchmark-assets`
- `synthesize-evidence` (when the unmet-need or pipeline framing turns on what is already approved and what the approved labels actually say)
- `validate-target` (when the disease primer needs target-level genetic support for key mechanisms)

## Output Contract

Return an indication research package that can include:

- disease context
- epidemiology or sizing assumptions
- current standard-of-care observations
- unmet need framing
- competitive pipeline observations
- evidence gaps and boundary conditions

## Core Research Pattern

For indication-level research, use a layered approach:

1. use structured MCP tools to define the baseline disease, population, and pipeline picture
2. use `search_documents`, `read_document`, and relevant recent-event checks to augment that picture
3. reconcile the final analysis before returning it

In particular, the competitive pipeline portion should not stop at `research_landscape` or other structured enumeration. Treat those outputs as the baseline set, then augment with documents to catch recent updates, missing programs, or evidence that materially changes interpretation.

When the approved-treatment landscape matters (it usually does for unmet-need framing), enumerate the approved drugs in the indication and anchor competitive language in primary-source label text rather than press releases: `search_documents(source_types=["fda_filings"])` scoped to each drug, then `read_document` with `format="sections"` to locate the approved-indication wording before pulling full text. Read labels only for the drugs you will actually cite.

For disease context, sizing, standard of care, and unmet need, do not rely on generic narrative retrieval alone when the user asks for specific quantitative claims. Build a small numeric checklist, retrieve sources for each high-signal number with `search_documents` and `read_document`, and only then synthesize the narrative.

When the request includes many specific rates, percentages, costs, time windows, or other figures, invoke `synthesize-evidence` in Numeric Claim Audit Mode before writing prose. The indication package should preserve the working claim table or summarize unresolved numeric gaps.

Keep the raw indication name in `research_landscape.indication` and move sponsor/company/disambiguating text into `context`. Apply the same pattern to any `research_entity` calls for named assets or companies referenced during the workflow.
- when the packet splits into independent evidence workstreams (epidemiology, standard of care, pipeline), run them in parallel with the `research-assistant` sub-agent, then reconcile the returned claim sets before synthesis

## Fallback Rules

- If the indication or a referenced entity fails to resolve, retry with the raw name and move extra identifying text into the optional context hint.
- If the pipeline baseline appears sparse, run secondary entity discovery and document augmentation before finalizing.
- If the narrative requires specific figures, run a checklist verification pass and preserve any missing figures as gaps rather than substituting adjacent numbers silently.
- If host web fetch is blocked, prefer `search_documents` and `read_document` rather than repeated blocked fetch attempts.
