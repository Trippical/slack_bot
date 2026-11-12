# Inputs from the workplace

People fill this in. The implementing agent reads it and asks about anything left blank. Do not put secrets here. Record only where each secret is stored.

If this file and `SPEC.md` disagree about what is needed, the spec wins.

## 1. Answers to the open items (SPEC §14)

| # | Question | Answer |
|---|---|---|
| 1 | Can CI reach production Postgres to publish catalog bundles? If not, publishing runs as a Kubernetes Job. | |
| 2 | Who is on call for the bot? Who staffs the analyst queue? Who in data governance approves and grants new source tables? | |
| 3 | Is there a separate dev Databricks warehouse for CI? | |
| 4 | Which model and provider does the router use: Databricks, Bedrock, or Fireworks? | |
| 5 | Which three real queries, on which tables, seed milestone 1? Paste them in section 4. | |
| 6 | Which environments exist? The spec assumes dev and prod, each with its own Slack app, Postgres, and service principal. | |
| 7 | Which CI platform, image registry, and rollout mechanism? Who owns them? | |

## 2. Identifiers and access

Fill in one column per environment. The "Goes in" column says where the value ends up.

| What | Goes in | Dev | Prod |
|---|---|---|---|
| Slack bot token | env `SLACK_BOT_TOKEN` | stored at: | stored at: |
| Slack app-level token with `connections:write` | env `SLACK_APP_TOKEN` | stored at: | stored at: |
| Pilot channel IDs | `config.yaml` `slack.allowed_channels` | | |
| Help channel ID | `config.yaml` `slack.help_channel` | | |
| Analyst queue channel ID, a private channel | `config.yaml` `slack.queue_channel` | | |
| Postgres connection string | env `DATABASE_URL` | stored at: | stored at: |
| Databricks workspace host | `config.yaml` `databricks.host` | | |
| Databricks SQL warehouse ID | `config.yaml` `databricks.warehouse_id` | | |
| Service principal client ID and secret | env `DATABRICKS_CLIENT_ID`, `DATABRICKS_CLIENT_SECRET` | stored at: | stored at: |
| `SELECT` grants on the seed tables for the service principal | Unity Catalog | granted by, on: | granted by, on: |
| Model endpoint and model name | `config.yaml` `router.endpoint`, `router.model` | | |
| Model credentials | env `LLM_API_KEY` or the provider's own | stored at: | stored at: |
| Kubernetes cluster and namespace | deploy manifests | | |
| Outbound network access to Slack, Databricks, and the model endpoint | network policy | confirmed by: | confirmed by: |
| Fiscal year start, first day of a month | `catalog/calendar.yaml` | | |

## 3. Slack app manifest

Create one app per environment from this manifest, then generate the app-level token by hand under Basic Information, App-Level Tokens, with the `connections:write` scope. Invite the bot to the pilot channels and to the queue channel.

This manifest was written from the spec and has not been tried. Correct it here if Slack rejects any part of it.

```yaml
display_information:
  name: Insight Bot
  description: Answers data questions from the curated insight catalog.
features:
  bot_user:
    display_name: insightbot
    always_online: true
oauth_config:
  scopes:
    bot:
      - app_mentions:read
      - chat:write
      - reactions:write
settings:
  event_subscriptions:
    bot_events:
      - app_mention
  interactivity:
    is_enabled: true
  socket_mode_enabled: true
  org_deploy_enabled: false
  token_rotation_enabled: false
```

## 4. Seed queries

Paste the three queries for milestone 1 here, one block each. For each one give:

- the question it answers, in the words an employee would use;
- what it does not answer, so the router knows when to stay away;
- the SQL as the analyst runs it today;
- the tables it reads;
- which values change from one request to the next, such as a period or a region, and the allowed values for each.

The remaining seed insights for the pilot, about 20 in all, come later through the paved path in SPEC §10.3.

## 5. People

| Role | Name or channel |
|---|---|
| Who the implementing agent asks when it is blocked | |
| Who checks each milestone's "Done when" | |
| Who approves production deploys | |
