---
name: synthesize-evidence
description: "Turn already-discovered facts into evidence-backed claims with evidence ratings, verbatim quotes, and canonical source references. Use as the final evidence pass before presenting findings, or when a claim must be tied back to a specific filing. This is a building block the workflow skills compose."
---

# Synthesize Evidence

This primitive is the bridge from research findings to defensible claims.

## Use When

- a claim needs stronger support
- a workflow has facts but not enough provenance
- you need to downgrade or tighten a statement based on what the documents actually say
- a narrative depends on quantitative claims such as incidence, prevalence, utilization, outcomes, cost, treatment windows, response rates, or safety rates

## Core Tools

- `search_documents`
- `read_document`
- `get_recent_events`
- `search_documents` with `source_types=["fda_filings"]` (when claim provenance is a regulator label)

## Output Contract

Return structured claim objects:

- `claim`
- `evidence_rating`
- `citations`
- `notes`
- `gaps`

Example citation object:

```json
{
  "source_ref": {
    "id": "doc_abc123",
    "kind": "document",
    "tool": "read_document",
    "url": "https://example.com/doc"
  },
  "quote": "verbatim supporting text"
}
```

## Numeric Claim Audit Mode

Use this mode when the output needs quantified disease burden, treatment utilization, standard-of-care rates, clinical outcomes, safety rates, pricing, market size, or other numeric claims.

Before writing prose:

1. Extract a checklist of required numeric claims from the user request.
2. For each claim, run targeted `search_documents` queries rather than one broad topic search.
3. Read the strongest source candidates with `read_document`, usually `format="sections"` or `format="citations"`.
4. Return a working claim table with:
   - `claim_needed`
   - `value_found`
   - `source_ref`
   - `quote`
   - `evidence_rating`
   - `status`: `found`, `adjacent`, `conflicting`, or `missing`
   - `notes`
5. Only synthesize numbers marked `found` or clearly `adjacent`; preserve `conflicting` and `missing` items as gaps.

Do not substitute a nearby number silently. For example, if the user asks for AIS share of all strokes and the source only supports global ischemic-stroke share, record the mismatch rather than treating it as the requested number.

## Quality Bar

- use verbatim supporting quotes
- prefer primary or authoritative documents when available
- use the actual document ID, NCT ID, or URL returned by the tool result; do not invent plugin-local aliases
- for regulatory claims (approval status, label content, exclusivity, designations), cite the document ID (`doc_...`) plus its date. Direct quotes from the label text are preferred over paraphrase: locate the filing with `search_documents(source_types=["fda_filings"])`, use `read_document` with `format="sections"` to find the relevant section, then pull `format="text"` narrowly for the passage you will quote
- if host web fetch is blocked, treat `search_documents` plus `read_document` as the standard fallback path
- downgrade evidence strength when the support is indirect
- if the support is missing, record the gap instead of forcing a claim
- do not write final narrative prose until high-signal numeric claims have been audited into structured claim objects
