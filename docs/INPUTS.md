# Inputs from the workplace

People fill this in. The implementing agent reads it and asks about anything left blank. Do not put secrets here. Record only where each secret is stored.

If this file and `SPEC.md` disagree about what is needed, the spec wins.

## 1. Answers to the open items (SPEC §14)

| # | Question | Answer |
|---|---|---|
| 1 | Can CI reach production Postgres to publish catalog bundles? If not, publishing runs as a one-off job inside the platform. | |
| 2 | Who is on call for the bot? Who staffs the analyst queue? Who in data governance approves and grants new source tables? | |
| 3 | Is there a separate dev Databricks warehouse for CI? | |
| 4 | Which model and provider does the router use: Databricks, Bedrock, Fireworks, or another OpenAI-compatible endpoint? | |
| 5 | Which three real queries, on which tables, seed the real-system check of milestone 1? Paste them in section 4. The offline build does not need them. | |
| 6 | Which environments exist? The spec assumes dev and prod, each with its own Slack app per bot, Postgres, bucket or prefix, config file, and service principal per credentials block. | |
| 7 | Which CI platform, image registry, container platform, and scheduler? Who owns them? | |
| 8 | Which bots exist at launch? One is a fine answer. Fill in one copy of section 2a for each. | |
| 9 | Which bucket holds cached answers, who may write to it, what is its lifecycle expiry, and which scheduler runs the cache refresh? Not needed before milestone 5. | |

## 2. Identifiers and access

Fill in one column per environment. The "Goes in" column says where the value ends up.

### 2a. Per bot

Copy this table once for each bot. `<BOT_ID>` is the bot id in upper case.

Bot id. It starts with a lowercase letter, then lowercase letters, digits, and underscores. It is the same in every environment and is never renamed later: 

| What | Goes in | Dev | Prod |
|---|---|---|---|
| Slack bot token | env `SLACK_BOT_TOKEN_<BOT_ID>` | stored at: | stored at: |
| Slack app-level token with `connections:write` | env `SLACK_APP_TOKEN_<BOT_ID>` | stored at: | stored at: |
| Allowed channel IDs | `config.yaml` `bots[].slack.allowed_channels` | | |
| Help channel ID | `config.yaml` `bots[].slack.help_channel` | | |
| Analyst queue channel ID, a private channel. Bots may share one. | `config.yaml` `bots[].slack.queue_channel` | | |
| Wording for the help, wrong-channel, and scope-note replies, if the defaults do not fit | `config.yaml` `bots[].texts` | | |
| Credentials block, if not `default` | `config.yaml` `bots[].credentials` | | |
| Which insights this bot should offer, if not all of them | `bots:` lists on those insights in the catalog | | |

### 2b. Shared

| What | Goes in | Dev | Prod |
|---|---|---|---|
| Postgres connection string | env `DATABASE_URL` | stored at: | stored at: |
| Databricks workspace host | `config.yaml` `databricks.host` | | |
| Databricks SQL warehouse ID | `config.yaml` `databricks.warehouse_id` | | |
| Service principal client ID and secret, once per credentials block | env `<env_prefix>DATABRICKS_CLIENT_ID`, `<env_prefix>DATABRICKS_CLIENT_SECRET` | stored at: | stored at: |
| `SELECT` grants on the seed tables, for every service principal whose bots offer those insights | Unity Catalog | granted by, on: | granted by, on: |
| Model provider, endpoint, and model name | `config.yaml` `router.provider`, `router.endpoint`, `router.model` | | |
| Model credentials | env `LLM_API_KEY` or the provider's own | stored at: | stored at: |
| Cache bucket and key prefix | `config.yaml` `cache.bucket`, `cache.prefix` | | |
| Cache store endpoint, if the store needs one | `config.yaml` `cache.endpoint` | | |
| Bucket access: read for the bot, write for the refresh tool only | the platform's identity or env credentials | granted by, on: | granted by, on: |
| Bucket lifecycle expiry, longer than the largest `max_age` | the object store | | |
| Scheduler and schedule for `tools/refresh_cache.py`. One run each day after midnight and after the source tables' daily load. | the adopter's scheduler | | |
| When the source tables' daily load finishes | the refresh schedule | | |
| Where the service runs | the adopter's deployment | | |
| Where each environment's config file lives | env `CONFIG_PATH` | | |
| Outbound network access to Slack, Databricks, the model endpoint, and the object store | network policy | confirmed by: | confirmed by: |
| Fiscal year start, first day of a month | `catalog/calendar.yaml` | | |

## 3. Slack app manifest

Create one app per bot per environment from this manifest, changing the name and display name for each bot. Then generate the app-level token by hand under Basic Information, App-Level Tokens, with the `connections:write` scope. Invite each bot to its allowed channels and to its queue channel.

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

Paste the three queries for the real-system check of milestone 1 here, one block each. For each one give:

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
