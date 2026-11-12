# Critique of the current Slack insight bot design

## What is right and worth keeping

- **Curated-only.** No free-form text-to-SQL. This is the single most important decision and it is correct. It bounds cost, eliminates hallucinated joins, and makes every answer attributable to a reviewed artifact.
- **Analyst fallback queue plus a paved path to add insights.** This is the growth engine. Unanswered questions become a prioritized backlog; analyst answers become catalog candidates. Everything else in the design should serve this loop.
- **Pre-approved parameterized Databricks queries + dashboard configs + cache.** Right set of answer types.

## Where the design is weak

### 1. Warm pool + pushed config is a job queue with a hand-built dispatcher
Clarified 2026-11-09: the pods are already warm, so there is no cold start. The remaining problem is the shape, not the latency. A pool of idle pods that each receive a config per question needs a dispatcher that knows which pods are idle, which are busy, and which died mid-question. That tracking state is where the bugs live.

Flip push to pull. Each pod runs a loop: claim one queued row from a Postgres `jobs` table with `FOR UPDATE SKIP LOCKED`, look up the catalog config it needs, do the work, post to Slack, mark the row done, repeat. The table tracks queued, running, and done; a sweeper returns rows stuck in running past a timeout to queued, so a dead pod's job is picked up by another. No dispatcher, no pod state.

Why Postgres and not Redis (decided 2026-11-09): a few hundred questions a day is under one per second. Redis earns its place at hundreds of jobs per second. Postgres already holds thread state, the catalog bundle, and the audit log, so the jobs table makes it one managed datastore instead of two. The job row doubles as the audit record.

Two more wins from the pull shape. The work is I/O bound (waiting on the Claude API and the Databricks warehouse), so one async worker handles 8 to 16 questions at once instead of one. And scaling has a real signal: KEDA's postgresql trigger scales the Deployment on the count of queued rows, which an idle pool cannot express. At this volume two fixed replicas is a fine MVP; add KEDA when the first backlog appears.

If "pods hosting models" means **self-hosted inference** (vLLM etc.), keep inference as a steady, separately scaled service. Workers call it like any other API.

### 2. Routing is hand-waved and it is the actual hard problem
Routing is three things: (a) pick the catalog entry, (b) extract and validate parameters, (c) decide between answer / clarify / escalate.

Cleanest mechanism: **expose every catalog insight as a tool definition** (name, description, example questions, JSON schema of params). One model call with the question, thread history, and the tool list. The model either calls one tool with typed params, calls `ask_clarifying_question`, or calls `escalate_to_analyst`. Strict schemas give you curated-only for free; invalid params fail before any SQL runs.

Once the catalog passes roughly 50-100 entries, add an embedding shortlist (top 10-20 by similarity over name + description + example questions) and only pass those tools.

Drop the "make another session with just the context" idea. Two sequential model calls is fine (route, then render results into prose). Two separate sessions or pods is not.

Make routing emit a confidence and use thresholds: high = answer, medium = ask one clarifying question in thread, low = escalate. Clarifying beats escalating for ambiguous but answerable questions.

### 3. Parameters are the security boundary
"Pre-approved query" is only safe if parameters are typed and bound, never interpolated. Use the Databricks SQL Statement Execution API with named parameters. Enums for dimensions, bounded date ranges, allowlisted values. Run under a read-only service principal scoped to specific Unity Catalog schemas, with statement timeouts and row limits.

### 4. Authorization (waived)
Scope decision 2026-11-09: the catalog only ever holds company-public insights; anything restricted is answered by a human. So no per-user entitlements are needed. Keep two cheap guards anyway: a channel allowlist, and a Unity Catalog tag on tables the service principal may read, so an analyst cannot accidentally onboard a restricted table.

### 5. Drop the answer cache for the MVP
A Databricks SQL warehouse already returns cached results for an identical statement with identical parameters, and serverless warehouses keep that cache across restarts. Measure its hit rate before building an app-level cache. If one is needed later, cache the **query result** keyed by (insight_id, version, normalized params), not the rendered prose, and invalidate when the upstream table advances rather than on a fixed TTL. Dashboards should be links with filters applied, not regenerated artifacts.

### 6. The escalation queue should produce catalog candidates, not just answers
An analyst's reply must carry the SQL they used and a one-click "promote to catalog" that opens the paved path with the question and SQL prefilled. Post the analyst's answer back into the original thread. Tell the asker the expected SLA at escalation time.

### 7. Paved path needs tests, and the tests are the example questions
Catalog as code in git: one directory per insight with `insight.yaml`, `query.sql`, `examples.yaml`, owner, freshness. PR-reviewed. CI should:
- dry-run the SQL against Databricks (EXPLAIN) with sample params
- reject any string interpolation of params
- run every example question across the whole catalog through the router and assert it routes to the right insight (routing regression test). Adding one insight must not steal questions from another.

The agentic coder's job is to scaffold that directory from an analyst's question + SQL, write the example questions, and open the PR.

### 8. No eval loop
Log every question -> routed insight -> confidence -> params -> cache hit -> latency -> thumbs reaction. Unrouted and thumbs-down questions are the analyst backlog, ranked by frequency. This is how you know routing accuracy is improving and what to build next.

### 9. Slack plumbing details
- Socket Mode avoids public ingress for an internal bot. Events API over HTTPS needs an ingress and the 3 s ack.
- "Certain threads" means a channel allowlist plus @mention; store thread_ts so follow-ups in the thread get prior turns as context.
- Treat message text as untrusted input to the router prompt. Prompt injection can only ever select an approved tool with validated params, which is another reason tool-use routing is the right shape.

## Alternatives

### A. Minimal monolith
Slack Bolt (Socket Mode) -> one stateless service -> tool-use router over catalog -> Databricks SQL API -> render -> reply. Postgres for catalog index, question log, escalation queue. One Deployment, two replicas. Ships in weeks, and the catalog-as-code repo is the only durable asset anyway.

### B. Gateway + workers over a Postgres jobs table (recommended)
Gateway Deployment -> `jobs` table -> worker Deployment (KEDA postgresql trigger on queued count) -> same router. Only slightly more than A, and it gives you retries, visibility into in-flight work, and independent scaling of ingestion and work. Nothing per-question is ever scheduled. No Redis.

### C. Databricks-native
AI/BI Genie space over curated tables with trusted/verified queries, fronted by a thin Slack app. Least code, but Genie generates SQL, which violates the curated-only constraint unless you lock it down to trusted assets only. Worth a spike to see whether "verified queries only" mode is strict enough; if not, it is still a good analyst-side exploration tool.

## Bottom line
Keep the product shape. Replace the pod-pool with a Postgres jobs table and stateless workers. Make routing a tool-use call over the catalog with strict schemas and a clarify/escalate branch. Put the catalog in git with example questions as routing tests. Rely on the warehouse result cache until measurement says otherwise. Add an eval loop on day one.
