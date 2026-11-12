# Your design vs. the independent design

Files: `critique-of-current-design.md` (my critique), `teammate-design.md` (independent from-scratch design, written without seeing yours), this file.

## Where both designs agree (high confidence these are right)
- Curated-only, no text-to-SQL. Only SQL merged to main ever executes.
- Reader/gateway acks Slack fast, hands off to a queue, workers do the work.
- Pre-approved parameterized Databricks queries with typed, bound parameters via the Statement Execution API. Never string interpolation.
- Result cache plus dashboards as parameterized deep links.
- Analyst escalation queue in Slack, with answers posted back to the original thread.
- Catalog as code in git, PR-reviewed, with an agentic coder scaffolding new insights.
- Example questions on each insight double as routing regression tests in CI.

## Where the independent design goes further than yours
| Topic | Your design | Independent design |
|---|---|---|
| Unit of work | Warm pod pool, dispatcher pushes a config to an idle pod per question | Stateless worker Deployment pulls from a queue, KEDA-scaled on backlog, ~8 concurrent tasks per pod. Recommended queue: Postgres jobs table with SKIP LOCKED, not Redis |
| Routing | Unspecified; maybe the worker model finds a config, maybe a second session | Hybrid retrieval (Postgres FTS + pgvector) to top-8, then one forced tool call; deterministic Python validates everything |
| Confidence | Not addressed | Thresholds in code (0.80 auto, 0.55-0.80 hedged, else clarify/escalate), recalibrated monthly |
| Parameters | "Pre-approved queries" | Typed param system: date_range, enum from UC dims, entity via lookup query; free-text string params banned |
| Authorization | Waived by scope: public insights only, restricted handled by humans | Three sensitivity tiers (not needed under current scope); keep only the UC tag gating what the service principal can read |
| Catalog quality | Not addressed | does_not_answer and negative_utterances fields, collision report on every PR, CODEOWNERS per domain |
| Cache | Fixed TTL implied | Redis result cache with freshness invalidation. Recommended: rely on the Databricks warehouse result cache for the MVP and measure |
| Eval loop | Not addressed | Thumbs with reason codes, "Not what I meant" yields labelled corrections, weekly eval PR, unrouted backlog clustering |
| Escalation -> catalog | "Analyst fills it out" | Claim/Answer/Duplicate buttons, SLAs, promotion board, PROMOTION.md brief handed to a /new-insight command |

## Where I would push back on the independent design
- **Redis is unnecessary at this volume.** A Postgres jobs table does the queue job and the warehouse result cache does the cache job. One managed datastore instead of two.
- **Five deployables is more than an MVP needs.** catalog-service, escalation-svc and admin-web can start as modules inside one service. Split later.
- **Two LLM calls per question (route + narrate)** is fine, but the narrative call is the one most exposed to prompt injection via result data. Its output checks are good; consider deterministic templates only for the MVP.
- **Hybrid retrieval from day one** is premature at 20 insights. Pass the whole catalog as tools until roughly 50-100, then add retrieval. The design's own CI metric (recall@8) is still worth wiring early so the switch is measurable.
- **Monthly threshold recalibration and a weekly eval PR** assume an analytics team with time to review. Good target, but phase 2.

## Recommendation
Adopt the independent design's routing, parameter typing, catalog schema, and security model essentially as written. Keep your product shape and the escalation-to-paved-path loop. Collapse the runtime to two deployables for the MVP: gateway and worker, with one managed Postgres off-cluster holding the jobs table, thread state, catalog bundle, and audit log. Drop the pod pool entirely. Skip Redis. Add KEDA's postgresql trigger when the first backlog appears.
