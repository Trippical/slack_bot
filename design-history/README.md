# Design history (background only)

Nothing in this folder is a build instruction. The source of truth is `SPEC.md` in the repo root.

These files record how the design was reached. They disagree with the current spec in places, and the spec wins every time.

| File | What it is | Where it is out of date |
|---|---|---|
| `SPEC-v1.0.md` | The previous, larger spec. Useful as a detailed reference for features that `SPEC.md` lists under "Later". | Two deployables, KEDA in the MVP, dynamic enum lists, SLA timers, grant-sync job, Slack history as thread context. Contains known defects fixed in v1.1 (bundle flip, retry without backoff, duplicate failure notices). |
| `critique-of-current-design.md` | Critique of the original warm-pod-pool design. | Suggests one tool per insight and an embedding shortlist. Neither is in the spec. |
| `teammate-design.md` | An independent from-scratch design ("Lantern"). | Recommends Redis Streams, five services, pgvector retrieval, LLM narration of results, three auth tiers. None are in the spec. |
| `comparison.md` | Side-by-side of the original design and the independent one. | Same as above. |
