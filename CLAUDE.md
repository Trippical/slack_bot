# Notes for the implementing agent

You are building the Slack insight bot described in `SPEC.md`. The spec says what to build. This file says how to work on it.

## Read first

1. `SPEC.md`, all of it, once, before writing code. It is the source of truth.
2. `docs/PROGRESS.md`. It records where the build stands. Read it at the start of every session and update it at the end, because nothing else carries over between sessions.
3. `docs/INPUTS.md`. It holds what people at the workplace have supplied: identifiers, access, and answers to the open questions.

Leave `design-history/` alone. It holds earlier designs that contradict the spec, including Redis, a separate gateway, autoscaling, and per-user authorization. Reading it is more likely to mislead you than help you. The one exception: when a person asks you to build something from Appendix B, the spec points to `design-history/SPEC-v1.0.md` for the older detailed design. Open it then, and only for that feature.

## What must stay true

These are the owner's decisions. They are not up for redesign. If one of them seems to block you, stop and ask.

- **Curated only.** The bot never runs SQL that is not a reviewed `query.sql` from the published bundle. The executor has one entry point that takes an insight id and validated parameters (§8). No SQL is built from strings, and nothing the model writes reaches SQL.
- **The model fills in a form and code decides.** One structured-output call per question. No tool loop and no agent framework. Every field is validated (§6.3) before anything acts on it.
- **Tiers never change what is executable.** Doc, then cache, then live, in that order (§8). The live tier is the only code that sends SQL anywhere. The cache holds rows that the live tier produced from the same reviewed query, and the refresh tool goes through the same executor. The router never sees or chooses a tier.
- **Doc insights are reviewed snippets.** They are posted as written. Do not search, summarise, or generate document text.
- **Bots are config, not forks.** A new bot is a config entry and a Slack app. No per-bot code, no per-bot branch, no per-bot deployment. A bot's catalog scope avoids confusion. It is not access control.
- **No per-user authorization, and no per-bot data isolation.** The catalog and the cache hold only data that any employee may see. Restricted questions go to people.
- **Mention only, Socket Mode, three scopes per Slack app.** Do not add scopes and do not read channel history.
- **Deployment-neutral.** `SPEC.md` and the code name no cloud, container platform, CI system, or company. Those belong in the adopter's own deployment notes, kept apart from the spec, and in `deploy/`, which holds examples only.
- **Keep it simple.** One service, two replicas, one Postgres, one bucket. Appendix B lists what was deliberately left out. Do not build any of it unless asked. That includes autoscaling, on-demand result caching beyond the grid in §8, and document retrieval. Do not add Redis, embeddings, or a second service either. Earlier designs had them and they were dropped.

## How to work

- Build the milestones in §13 in order, one at a time. Each has two bars: "done on fakes" and what the real systems must show. Reach the first in offline mode (§10.4) with the sample catalog and a real Postgres. Show that it holds by running it, report what you ran and what you saw, then stop so a person can check before you start the next one.
- You can go a long way without credentials. The repo half of milestone 0 and the "done on fakes" bar of milestones 1 to 5 need only Postgres. Do not wait on `docs/INPUTS.md` for those.
- The other half of milestone 0 is other people's work. Check `docs/INPUTS.md`. If something you need for a real-system check is blank, ask for it. Do not invent identifiers, credentials, or seed queries.
- The language is Python with asyncio, as in the spec's code. For libraries, CI, the image registry, and deployment, follow the workplace's standards and record what you chose in `docs/PROGRESS.md`.

## When the spec and reality disagree

The spec was reviewed five times by reading. Nothing in it has been run. Expect some details to be wrong, most likely in how Slack, Databricks, or the model API actually behave.

- **A fact about an external API is different**, such as a field name, an error code, or a limit. Follow reality, correct `SPEC.md` in the same change, and note it in `docs/PROGRESS.md`.
- **Behaviour would change**, meaning what a user or analyst sees, what is stored, or how a failure is handled. Stop and ask.
- **The spec is silent.** Choose the simplest thing that fits the rest of the spec, and log the choice in `docs/PROGRESS.md` so a person can overrule it.

Keep `SPEC.md` true. Code and a spec that have drifted apart are the worst outcome for whoever comes next.

§14 has a "Verify during the build" list. Check each item when you first write code that depends on it, and record the result in `docs/PROGRESS.md`.

## Testing

- Most of the bugs found in review were in the failure paths: the worker loop, the sweeper, the failure notifier, pick handling, and `ensure_escalation`. They depend on Postgres locking and transaction behaviour, so test them against a real Postgres, not a mock.
- Slack, Databricks, the model, and the object store have fakes (§10.4), switched on together by one flag. Postgres is never faked. A milestone is done only when its real-system column in §13 has been shown. Say plainly which results came from fakes.
- The fake router returns fixture decisions. A green routing eval on the sample catalog proves the harness, not a model.
- The example and negative utterances in the catalog are the routing tests. Never put them in the router prompt. Doing so would grade the model on questions it has already seen.

## Safety

- Use dev credentials and the dev Slack apps only. Never connect a development process to a production Slack app. Slack delivers each event to one connection, so that process would take real questions.
- Never point a development process or a test at the production bucket. The refresh tool writes what the bot will post.
- Secrets live in environment variables. Never write them to the repo, `config.yaml`, logs, or `docs/PROGRESS.md`.
- Do not deploy to production, publish a bundle to production, or change table grants unless a person says so.
