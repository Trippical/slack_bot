# Build progress

The implementing agent keeps this file current. Read it at the start of every session and update it before stopping. Write for a reader who has none of your context: what is done, what was run to prove it, and what comes next.

## Milestones (SPEC §13)

Status is one of: not started, in progress, done on fakes, done on real systems, checked by a person.

| # | Milestone | Status | Evidence: what was run and what happened | Checked by, on |
|---|---|---|---|---|
| 0 | Prerequisites | not started | | |
| 1 | Walking skeleton | not started | | |
| 2 | Jobs table | not started | | |
| 3 | Clarify and picks | not started | | |
| 4 | Escalation | not started | | |
| 5 | Catalog pipeline | not started | | |
| 6 | Pilot | not started | | |

## Next step

One or two sentences on exactly where to pick up.

## Verify during the build (SPEC §14)

Check each item when you first write code that depends on it. Record what you ran and what you saw. If the spec was wrong, fix `SPEC.md` in the same change and log it below.

| Item | Result | How it was checked, and when |
|---|---|---|
| The chosen model returns schema-valid structured output at temperature 0 | | |
| An omitted parameter `value` binds `NULL` on the Statement Execution API | | |
| The warehouse result cache hits for an identical statement with identical bound parameters | | |
| The `view_submission` payload carries `view.id` and `private_metadata` | | |
| Socket Mode needs only `connections:write` on the app-level token | | |
| The Slack error codes for a deleted thread and a deleted message, and the passing error codes named in §5.3 | | |
| `chat.getPermalink`, `views.open`, and `chat.postEphemeral` work with the three scopes in §3 | | |
| How many times, and how quickly, Slack retries an unacknowledged event over Socket Mode | | |
| `(:region IS NULL OR region = :region)` runs with a typed `NULL` parameter | | |

## Choices made where the spec was silent

A person can overrule any of these.

| Date | Choice | Why | Where in the code |
|---|---|---|---|
| | | | |

## Changes made to SPEC.md

| Date | Section | What changed | Why |
|---|---|---|---|
| | | | |

## Questions waiting on a person

| Date | Question | Blocking which milestone | Answer |
|---|---|---|---|
| | | | |
