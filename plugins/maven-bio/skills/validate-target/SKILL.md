---
name: validate-target
description: "Validate the genetic basis of a drug target: gene-disease association strength, gene constraint, tractability, and known drugs, from public biomedical databases. Use when a thesis turns on whether a target is genetically validated."
---

# Validate Target

This primitive answers "is this target genetically validated, and how strongly" with evidence from public databases (Open Targets, gnomAD, GWAS Catalog).

## Use When

- the thesis or landscape turns on a target's genetic validation
- a workflow needs gene-disease association strength before ranking programs
- you need target tractability / druggability or gene constraint (LOEUF)
- you need the drugs in development against a target, or repurposing hypotheses (speculative)

## Core Tools

- `match_entity` - confirm the gene / disease / target is the one you mean, and pick up its canonical name and synonyms. Use the resolved *name*, not the returned `tgt_` id, when calling `research_bio_evidence`.
- `research_bio_evidence(name, entity_type, aspects)` - the bio evidence tool. Select `aspects`:
  - `tractability` - target druggability
  - `constraint` - gene constraint (gnomAD LOEUF)
  - `associations` - gene<->disease association (genetic + L2G + overall score); pass `context=<disease>` to scope a target to one disease
  - `known_drugs` - drugs and clinical candidates in development against the target
  - `repurposing` - repurposing candidates (opt-in; always speculative)
- `research_entity`, `search_documents`, `read_document` - strengthen or cross-check bio findings against the Maven corpus

## Output Contract

Return a structured target-validation object that can include:

- resolved target (gene symbol) and disease (EFO/MONDO id)
- association strength with `evidence_rating` (Direct for curated DB associations, Indirect for inferred/aggregated scores)
- gene constraint and tractability with `evidence_rating`
- drugs in development against the target
- repurposing hypotheses, each explicitly marked speculative
- explicit gaps (unresolved ids, fuzzy disease-ontology matches, no association found)

Each claim carries: `claim`, `evidence_rating`, `citations` ([{source_id, quote}]), optional `notes`. Evidence is structured metadata attached to claims, not inline prose markup.

## Quality Bar

- a fuzzy disease-ontology match downgrades the evidence_rating one level and is flagged
- repurposing output is always labeled speculative; never Direct Evidence
- a curated database association (with a source record) is Direct; an inferred score is Indirect
- do not treat an entity-resolution result as a citation
- pass `research_bio_evidence` a raw gene symbol or disease name (`PCSK9`, `NASH`), never a Maven `tgt_`/`ind_` id. It resolves against external databases, which do not know Maven ids, and an id silently returns `resolved: false` with no error rather than failing loudly. For a target-disease pair, put the disease in `context`
- check `resolution.resolved` on every `research_bio_evidence` response before reading the payload. When it is false, retry once with an official gene symbol or a documented synonym from `match_entity` before concluding anything
- report an unvalidated target only when a resolved lookup came back with no association. `resolved: false` means the name did not match, not that the target lacks evidence; the two must never be reported the same way
