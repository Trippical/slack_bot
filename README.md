# Slack Insight Bot

A Slack bot that answers employee questions only from a curated catalog of approved, parameterized Databricks queries and dashboard links. Anything it cannot answer goes to a human analyst queue. It never writes its own SQL.

## Start here

- [SPEC.md](SPEC.md) is the build specification, v1.1. It is the source of truth. Section 13 is the build order and section 14 lists the open items.
- [CLAUDE.md](CLAUDE.md) tells the implementing agent how to work: what must stay true, how to handle gaps in the spec, and how to test.
- [docs/INPUTS.md](docs/INPUTS.md) is the form the workplace fills in: answers to the open items, identifiers, access, and the seed queries.
- [docs/PROGRESS.md](docs/PROGRESS.md) is the build log the implementing agent keeps between sessions.
- [design-history/](design-history/) holds earlier drafts. They are background only and the spec wins wherever they disagree.

## Status

Design only. No code has been written yet, and nothing in the spec has been run against real systems.
