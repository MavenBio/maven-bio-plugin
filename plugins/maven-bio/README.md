# Maven Bio Plugin

The Maven Bio plugin is a primitive-first research and evidence layer for BioPharma
intelligence workflows. It supports investment teams, pharma BD and corp dev, and in-house
portfolio strategy across drug pipelines, clinical trials, regulatory events,
public-company financials, and private financing rounds.

## Requirements

An active Maven Bio account. The plugin connects to the Maven Bio MCP connector at
`https://mcp.mavenbio.com`, which is read-only and scoped to your own account and
organization. Authentication uses OAuth; no API keys or passwords are shared with your
client.

- Documentation: https://mavenbio.com/mcp/docs
- Privacy policy: https://mavenbio.com/mcp/privacy
- Support: support@mavenbio.com

## What it provides

- research primitives under `skills/` covering competitive pipelines, indication
  landscapes, asset and company profiles, market sizing, deal and financing activity,
  commercial analogs, and evidence synthesis
- approval, label, and exclusivity analysis anchored in primary FDA and SEC filings
  through document search and read
- a specialized research sub-agent under `agents/`
- MCP connectivity via `.mcp.json`

## Design principles

- structured MCP tools define the baseline universe
- document search and reads augment and reconcile master analyses
- evidence is attached as structured data, not custom citation markup
- canonical source references are used instead of `srcN` aliases
- gaps are surfaced explicitly rather than smoothed over
- Claude built-in sub-agents are preferred over async task tools for interactive
  workflows
- the plugin does not ship hooks, custom runtime state, or artifact-format opinions; those
  belong to the host environment or the calling workflow

## Instruction model

Runtime behavior comes from the supported plugin surfaces:

- `skills/*/SKILL.md`
- `agents/*.md`
- MCP tool descriptions served by the connector
- `.claude-plugin/plugin.json`

## Changelog

### 0.6.0

- Approval, label, and exclusivity questions now route through document search and read
  against primary FDA and SEC filings rather than through dedicated regulatory tools.
- Analysis scaling guidance: when a set is larger than the evidence budget, narrow the set
  rather than thinning the evidence gathered per row.
