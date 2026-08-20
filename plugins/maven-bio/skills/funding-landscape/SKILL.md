---
name: funding-landscape
description: "Map capital flow across a space: who is funding an indication, mechanism, target, or investor set, how much, and how it has moved over time. Use for market-level money questions, for example 'who is funding TL1A', 'how much has gone into obesity'. Use trace-financings when someone wants the round-by-round record for one company, and competitive-pipeline for drug programs rather than capital."
---

# Funding Landscape

This workflow produces a capital-flow intelligence package for a space, not a drug-program landscape.

## Use When

- an investment team wants recent funding activity in an indication, mechanism, or target
- a BD or corp dev team wants the round-size and investor-mix picture for a space they are evaluating
- a portfolio strategist wants to gauge capital momentum, sentiment, or new-entrant pressure in a competitive area
- the question is about money flowing into a space, not about which drugs are in development there

For the drug-program-centric question (which assets are in trials, at what phase, with what mechanism), use `competitive-pipeline` instead.

## Primary Primitives

- `enumerate-entities` to scope the right indication, mechanism, target, or investor set
- `trace-financings` for round-level capital activity
- `synthesize-evidence` for any quantitative claim that will be cited (totals, top-investor lists, deal cadence)

## Optional Primitives

- `trace-events` for post-round announcements, deals, or regulatory milestones tied to recipients
- `profile-entity` for a focused dive on a notable recipient or investor surfaced by the round set

## Output Contract

Return a funding intelligence package that can include:

- scope summary (which indication / mechanism / target / investor set the package covers, and the time window)
- total raised in window (USD), with the confirmed and unconfirmed slices reported separately
- round count and round-type distribution (Pre-Seed through IPO and beyond)
- top recipients by total raised, each with round count and most recent round
- top investors by activity in the space (count of rounds participated, lead-investor signal where derivable)
- deal cadence (rounds per quarter or per month) so the reader can see acceleration or cooling
- notable rounds (top by value, oversubscribed, strategic investor presence, post-IPO follow-on)
- market signal interpretation (acceleration vs cooling, sub-mechanism momentum, investor concentration)
- explicit gaps and boundary conditions

## Aggregation Shortcuts

When the question is about distributions, rankings, or trends rather than individual rounds, prefer `aggregate_records(entity_type="financing", ...)` over enumerate-and-count. Canonical calls:

- Round-type distribution: `query="rounds in {scope} by financing type with total value and count"`
- Deal cadence: `query="rounds in {scope} by quarter since {date}"`
- Top investors: `query="rounds in {scope} grouped by investor, top 25"`

The `investors` field is M2M-exploded (one round with three investors counts in all three groups). `total_value_input_count` separates confirmed-value rounds from unconfirmed. Use `get_financings` for the round-level evidence that backs any claim you cite.

Government grants (NIH, NCI, NIAID, Innovate UK, Bpifrance) will top investor-count rankings if not filtered. For BD/corp dev personas, exclude grant funders or report them in a separate slice.

## Core Research Pattern

1. **Scope** the right ontology entry. Resolve the indication / mechanism / target (or, for a broad area, a top-level indication) with `match_entity`, or use the recipient-side filters (indication / modality / mechanism / target) surfaced through `search_entities("financing", ...)`. Define the time window explicitly. Decide whether the scope is recipient-side, investor-side, or both.
2. **Enumerate** the round set. For structured filters use `get_financings` directly; for ontology-anchored or fuzzy queries use `search_entities("financing", ...)`. Page through the full matching set, not a sample. For the aggregate shape (round-type distribution, deal cadence, top investors), use `aggregate_records` to get the picture in a single call before drilling into individual rounds.
3. **Augment** with documents and events. For high-signal rounds (top by value, strategic investor, recent), pull supporting `document_ids` via `read_document`, and catch newer rounds with `get_financings` scoped to the company (financing recency is not in `get_recent_events`).
4. **Reconcile** before finalizing. Compare round count and total raised against any prior expectation or anchor knowledge. If counts look suspiciously low, broaden the scope or rerun with a sibling ontology axis (parent indication instead of subtype). If they look noisy, separate the confirmed-value slice from the unconfirmed slice.

## Workflow Rules

- treat the unconfirmed-value slice as a separate visible bucket; never aggregate `total_value_usd: null` rounds into "total raised" without a footnote
- when reporting deal cadence, anchor on quarter-ending dates and call out the trailing-quarter window explicitly
- when reporting top investors, distinguish lead-investor presence (derivable from round documents) from participating-investor presence (the raw investor list)
- pair recent funding cadence with `get_recent_events` so the reader sees both the round and any post-round announcements (clinical readouts, deals, FDA actions) that contextualize the capital
- for VC and corporate venture investor lookups, the natural axis is `aspects=["investments"]` on `research_entity`, not the recipient-side filters
- do not present a funding landscape as proof of clinical or regulatory progress; the two layers are independent and the reader needs both
- distinguish private capital (financing rounds) from public-market signals (market cap, cash, runway from financial statements). The latter lives on `get_financials`; mention it when comparing public-comp valuations
- when the user is a BD or corp dev team sizing acquisition targets, surface a runway-implication slice: companies with thin recent funding, an old last round, or an aging Series B+ are candidate signals (combine with `get_financials` on public comps for full posture)
- when the picture needs per-company round histories or investor profiles, gather them in parallel with the `research-assistant` sub-agent, then reconcile the returned claim sets before aggregating

## Fallback Rules

- if the indication fails to resolve, retry with the raw name and move disambiguating text into the optional context hint; if it still fails, broaden to the parent indication and note the scope drift
- if `get_financings` returns zero rounds for a structured query, retry the same intent through `search_entities("financing", ...)` and check whether SSF resolves the scope differently before concluding no activity exists
- if the round set is suspiciously sparse, run sibling-axis enumeration (parent indication, broader mechanism class) and preserve the broader set as discovery scaffolding before finalizing
- if the round set is large and noisy, partition into core confirmed rounds (with supporting documents read in-session) and watchlist rounds (structured-only or thin documents) rather than collapsing every round into the headline narrative
- if a notable round's investor list is dominated by stub companies (`display_only=True` thin profiles), preserve the asymmetry; do not promote a stub-only round to the same evidence tier as a fully-resolved round
- if host web fetch is blocked, augment with `search_documents` plus `read_document` against round-linked documents rather than repeated blocked fetches
- if a workflow gap remains after the above (e.g., the user wants a filter that does not exist on `get_financings`), record it explicitly in the output rather than silently approximating it with a filter that means something else

## Persona Notes

- **Investment teams** typically want recent activity in a thesis space, lead-investor signal, and round-size distribution. Default the time window to trailing 12-24 months and lead with notable rounds plus deal cadence.
- **Pharma BD / corp dev** typically want acquisition-target screening or deal-target identification. Combine the recipient-side round set with `get_financials` on public comps and a runway-implication slice. Surface candidates with thin or aging funding as signal.
- **In-house pharma portfolio / strategy** typically want competitor capital posture and new-entrant detection. Anchor the scope on the same mechanism, target, or indication their internal asset operates in, and pair with `competitive-pipeline` for the program-level view.
