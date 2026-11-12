# Slack Insight Bot

A Slack bot that answers employee questions only from a curated catalog of approved, parameterized SQL queries against the company's data warehouse, reviewed text snippets, and dashboard links. One deployment can run several bots, each a config entry with its own Slack app and slice of the catalog. Anything it cannot answer goes to a human analyst queue. It never writes its own SQL. The warehouse sits behind one small adapter, so the design does not depend on which SQL engine holds the data.

## Start here

- [SPEC.md](SPEC.md) is the build specification, v1.3. It is the source of truth. Section 13 is the build order and section 14 lists the open items.
- [CLAUDE.md](CLAUDE.md) tells the implementing agent how to work: what must stay true, how to handle gaps in the spec, and how to test.
- [docs/INPUTS.md](docs/INPUTS.md) is the form the workplace fills in: answers to the open items, identifiers, access, and the seed queries.
- [docs/PROGRESS.md](docs/PROGRESS.md) is the build log the implementing agent keeps between sessions.
- [design-history/](design-history/) holds earlier drafts. They are background only and the spec wins wherever they disagree.

## Status

Design only. No code has been written yet, and nothing in the spec has been run against real systems. The spec is written so that the whole bot can be built and tested offline, on fakes and a sample catalog, before any credentials exist.
