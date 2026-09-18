---
name: harn-triggers-orchestration
description: Wire Harn to external events — [[triggers]] in harn.toml, cron/webhook/GitHub/Slack/Linear/Notion connectors, trigger handlers (SpawnToPool, ReminderInject, worker://, a2a://), agent pools/channels, and running `harn orchestrator serve`. Use when a Harn project reacts to events, runs on a schedule, or authors a connector package.
---

# Harn triggers, connectors and the orchestrator

**Triggers vs reminders:** triggers *spawn or schedule tasks* from external or
timed events; system reminders *modify a running session* by injecting typed
context (use the `ReminderInject` handler for that).

## Declare triggers in harn.toml

```toml
[package]
name = "review-bot"

[exports]
handlers = "lib.harn"

[[triggers]]
id = "github-prs"
kind = "webhook"
provider = "github"
match = { path = "/hooks/github", events = ["pull_request.opened"] }
handler = "handlers::on_pull_request"
when = "handlers::should_handle"          # optional predicate
dedupe_key = "event.dedupe_key"
retry = { max = 7, backoff = "svix" }
budget = { daily_cost_usd = 5.00, max_concurrent = 4 }
secrets = { signing_secret = "github/webhook-secret" }

[[triggers]]
id = "weekday-digest"
kind = "cron"
provider = "cron"
match = { events = ["cron.tick"] }
schedule = "0 9 * * 1-5"
timezone = "Europe/Madrid"
handler = "handlers::send_digest"
concurrency = { max = 1 }
```
`handler` may be `module::fn`, or a URI: `worker://<queue>`, `a2a://...`,
`eval_pack://<name>`. Budgets accept `on_budget_exhausted = "retry_later"`.

## Register at runtime

```harn
import { SpawnToPool, ReminderInject } from "std/triggers"
import { pool_create } from "std/lifecycle/pool"

fn configure(harness: Harness) {
  pool_create(harness.agent, {name: "pr-review-pool", max_concurrent: 4})
  harness.runtime.trigger_register({
    kind: "issue.opened",
    provider: "github",
    match: {events: ["issue.opened"]},
    handler: SpawnToPool({
      pool: "pr-review-pool",
      priority_from: "headers.priority",
      task_factory: { event -> { -> review(event) } },
    }),
  })
  harness.runtime.trigger_register({
    kind: "channel.emit",
    provider: "channel",
    match: {events: ["channel:pr.merged"]},
    handler: ReminderInject({target: "current", body: "PR merged — consider a patch release.", ttl_turns: 1}),
  })
}
```

## Providers and connectors

Built-in providers: `webhook`, `cron`, `email`, `kafka`, `nats`,
`postgres-cdc`, `pulsar`, `websocket`, `a2a-push`.
First-party connector packages: `harn-github-connector`, `harn-slack-connector`,
`harn-linear-connector`, `harn-notion-connector`, `harn-gitlab-connector`,
`harn-forgejo-connector`, `harn-gitea-connector`, `harn-bitbucket-connector`,
`harn-circleci-connector`, `harn-buildkite-connector`, `harn-sourcehut-connector`,
`harn-svn-connector`. Credentials: `harn connect` (guided OAuth setup).

### Authoring a connector (contract v1)
A pure-Harn connector package exports `provider_id()`, `kinds()`,
`payload_schema()`, and optionally `normalize_inbound()`, `poll_tick()`,
`call()`, `init()`, `activate()`, `shutdown()`.
**`normalize_inbound` must be pure** — no network, no LLM calls, no filesystem.
Add fixtures and verify before shipping:
```toml
[connector_contract]
version = 1

[[connector_contract.fixtures]]
provider = "slack"
name = "url verification"
kind = "webhook"
headers = { "content-type" = "application/json" }
body_json = { type = "url_verification", challenge = "challenge-token" }
expect_type = "immediate_response"
expect_event_count = 0
```
Run `harn package verify`. Scaffold with `harn new my-connector --template connector`.

## Running

- `harn orchestrator serve` — long-running process that owns triggers,
  queues, DLQ (`trigger.dlq`), backpressure, hot reload of `harn.toml`,
  metrics/OTel.
- `harn mcp serve` — expose the orchestrator as an MCP server.
- Deploy templates: Render (`deploy/render/render.yaml`), Fly.io
  (`deploy/fly/fly.toml`), Railway (`deploy/railway/railway.json`).

## Doc URLs
- Triggers quickref: https://harnlang.com/docs/llm/harn-triggers-quickref.md
- Triggers: https://harnlang.com/triggers.md — manifest: https://harnlang.com/triggers/manifest.md
- Budgets: https://harnlang.com/triggers/budgets.md — event schema: https://harnlang.com/triggers/event-schema.md
- Dispatcher: https://harnlang.com/triggers/dispatcher.md — webhook intake: https://harnlang.com/triggers/webhook-intake.md
- Trigger stdlib: https://harnlang.com/stdlib/triggers.md
- Agent channels: https://harnlang.com/agent-channels.md — pools: https://harnlang.com/agent-pools.md
- Orchestrator: https://harnlang.com/orchestrator.md (hot-reload, dlq, backpressure, worker-dispatch, secrets, multi-tenant, oauth under /orchestrator/)
- Connector catalog: https://harnlang.com/connectors/catalog.md — authoring: https://harnlang.com/connectors/authoring.md — testkit: https://harnlang.com/connectors/testkit.md
- Cron: https://harnlang.com/connectors/cron.md — GitHub: https://harnlang.com/connectors/github.md — Slack: https://harnlang.com/connectors/slack-events.md — Linear: https://harnlang.com/connectors/linear.md — Notion: https://harnlang.com/connectors/notion.md — Webhook: https://harnlang.com/connectors/webhook.md
- Deploy: https://harnlang.com/deploy/render.md, https://harnlang.com/deploy/fly.md, https://harnlang.com/deploy/railway.md
