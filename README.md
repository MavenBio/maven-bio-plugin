# Maven Bio plugin marketplace

Distribution repository for the **Maven Bio** Claude Code plugin, an evidence-backed
BioPharma research layer over Maven Bio's drug, trial, company, and deal knowledge graph.

## Install

```
/plugin marketplace add MavenBio/maven-bio-plugin
/plugin install maven-bio@maven-bio
```

An active Maven Bio account is required. The plugin connects to the Maven Bio MCP
connector at `https://mcp.mavenbio.com`, which is read-only and scoped to your own account
and organization. Authentication uses OAuth; no API keys or passwords are shared with your
client.

- Documentation: https://mavenbio.com/mcp/docs
- Privacy policy: https://mavenbio.com/mcp/privacy
- Support: help@mavenbio.io

## What it provides

Sixteen composable research primitives for competitive pipelines, indication landscapes,
asset and company profiles, market sizing, deal and financing activity, commercial analog
identification, and evidence synthesis, each grounded in primary FDA, SEC, clinical, and
press sources, plus a specialized `research-assistant` sub-agent.

## Layout

```
.claude-plugin/marketplace.json   marketplace manifest
plugins/maven-bio/                 the plugin (built for the directory profile)
assets/                            listing icons (512 / 1024 px)
```

The plugin under `plugins/maven-bio/` is a built artifact produced from the source tree in
`MavenBio/maven-backend` (`maven-bio-plugin/`) via
`scripts/build_cowork_bundle.py --profile directory`. Do not hand-edit it here; change the
source and rebuild.
