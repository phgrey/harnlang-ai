---
name: harn-llm-agents
description: Build LLM calls and agents in Harn — harness.llm.call, structured output, tools (tool_registry/tool_define), agent_loop and presets, sub-agents, MCP servers, sessions, compaction, providers. Use when a Harn program calls a model or runs an agent loop. Pair with harn-language for general syntax.
---

# Harn LLM calls and agents

Principle from the docs: **use the smallest API that matches the job**, and
climb only when you need more control.

| Need | API |
|---|---|
| One request → one response | `harness.llm.call(prompt, system, options)` |
| Validated JSON / typed result | `harness.llm.call_structured(prompt, schema, options)` |
| Multi-turn with tools until done | `agent_loop(harness, task, system, AgentSpec)` |
| Isolated child worker | `sub_agent_run(...)`, `spawn_agent(...)` + `wait_agent(...)` |
| Long-lived assistant | `daemon_spawn(...)` / `agent_loop` with `daemon: true` |
| Typed multi-stage pipeline | `workflow_execute` (see workflow-runtime.md) |

Pre-1.0: confirm option names against `references/agent-loop.md` or the live
docs before relying on anything unusual. Use `provider: "mock"` in tests and
examples — it's deterministic and needs no credentials.

## Single call

```harn
import { LlmCallOptions } from "std/llm/options"

fn main(harness: Harness) {
  const options: LlmCallOptions = {provider: "anthropic", model: "claude-sonnet-5", max_tokens: 256}
  const r = harness.llm.call("Translate 'Hello' to French.", "You are a concise translator.", options)
  harness.stdio.println(r.text)          // also r.visible_text, r.usage.input_tokens, r.outcome.kind
}
```

Key options: `provider` (`"auto"` infers from model prefix: `claude-*`,
`gpt-*`, `local:`, `ollama:`…), `model`, `max_tokens`, `temperature`,
`effort` (`low|medium|high`), `timeout_ms`, `output` (`"text"`, `"json"`, a
schema/type, or `{schema, strict, validation: "error", stream_abort}`),
`schema_retries`, `messages`, `tools`, `tool_search: "bm25"`.
OpenAI Responses API: `{provider: "openai", model: "gpt-5.4", api_mode: "responses", provider_tools: [{type: "web_search"}]}`.

## Structured output (preferred: schema-as-type)

```harn
type GraderOut = {verdict: "pass" | "fail" | "unclear", summary: string}

const out: GraderOut = harness.llm.call_structured(prompt, GraderOut, {
  provider: "auto",
  system: "You are a strict grader.",
})
```
Non-throwing variants: `call_structured_safe` (`r.ok`, `r.error.category`) and
`call_structured_result` (supports `repair: {enabled: true, model: "cheapest_over_quality(low)"}`).
Recover JSON from prose: `harness.llm.recover_schema(text, schema)`.

## Tools

```harn
import { ToolRegistry } from "std/tools"

pub fn with_search_tool(registry: ToolRegistry) -> ToolRegistry {
  return tool_define(registry, "search", "Search the corpus", {
    parameters: {query: {type: "string"}},
    returns: {type: "string"},
    handler: { args -> "hits for " + args.query },
  })
}

let tools = with_search_tool(tool_registry())
```
- JSON-schema params (`enum`, `minLength`, `items`…); optional `returns`.
- `executor`: default in-Harn handler, or `"host_bridge"` (+ `host_capability`),
  `"mcp_server"` (+ `mcp_server`), `"provider_native"`.
- `defer_loading: true` + `tool_search: "bm25"` for progressive disclosure;
  `namespace: "ops"` for grouping.
- Return typed facts with `agent_tool_handler_result(text, data)` from `std/agent/tool_lifecycle`.
- Ready-made: `agent_host_tools`, `agent_command_tools` (`std/agent/host_tools`),
  `git_tools` (`std/git`) — always restrict with `enabled_tools` /
  `allow_argv_prefixes` / `command_policy`.

## Agent loop

```harn
import { agent_loop } from "std/agent/loop"
import { AgentSpec } from "std/agent/options"

fn main(harness: Harness) {
  const opts: AgentSpec = {
    provider: "openai",
    model: "gpt-5-mini",
    tools: tools,
    tool_format: "native",
    loop_until_done: true,
    iteration_budget: {mode: "adaptive", initial: 4, max: 16, extend_by: 2},
    require_successful_tools: ["search"],
  }
  const result = agent_loop(harness, task, "You are a careful assistant.", opts)
  harness.stdio.println(result.status)   // "done", "completion_unverified", "stuck", "budget_exhausted", ...
  harness.stdio.println(result.text)
}
```
- **Only `status == "done"` proves completion.** Handle every other terminal
  status explicitly; don't treat `result.text` as success on its own.
- Presets: `agent_preset("audit" | "repair" | "summary" | "verify" | ..., overrides)`
  from `std/agent/presets`, then `agent_loop(harness, task, opts?.system, opts)`.
- Completion control: `verify_completion`, `verify_completion_judge`,
  `turn_end_condition`, `stop_after_successful_tools`, `require_artifacts`.
- Context: `compaction: {strategy: "hybrid", keep_last_n: 10}`, `auto_compact`,
  `context_callback`, `history: [...]`.
- MCP: `mcp_servers: [{name, transport: "http", url}, {name, transport: "stdio", command, args}]`.
- Streaming: `on_delta: { delta -> ... }`.

## Resilience (retry/fallback/budget)

`llm_retries` was removed in 0.10. Compose middleware instead:
```harn
import { default_llm_caller } from "std/llm/caller"
import { with_retry, compose } from "std/llm/handlers"

const caller = compose([with_retry({max_attempts: 4, base_ms: 250, backoff: "exponential"})])(default_llm_caller())
agent_loop(harness, task, system, {loop_until_done: true, llm_caller: caller})
```
Batch safely: `parallel settle items with { max_concurrent: 4 } { x -> harness.llm.call(...) }`.
Throughput limits: `rpm` in `harn.toml` or `HARN_RATE_LIMIT_<PROVIDER>=N`.

## Sub-agents

```harn
const result = sub_agent_run("Find the config entrypoints.", {
  provider: "mock",
  tools: repo_tools(),
  allowed_tools: ["search", "read"],
  token_budget: 1200,
  returns: {schema: {type: "object", properties: {paths: {type: "array", items: {type: "string"}}}, required: ["paths"]}},
})
if result.ok { harness.stdio.log(result.data.paths) } else { harness.stdio.log(result.error.category) }
```

## Providers

Env keys: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`,
`OPENROUTER_API_KEY`, `GROQ_API_KEY`, `DEEPSEEK_API_KEY`; Ollama/llama.cpp/MLX/vLLM
local. Check with `harn doctor --check-providers`, `harn models list`,
`harn models test <model> --provider <p>`. Query capabilities at runtime:
`harness.llm.provider_capabilities(provider, model)`.

## References

- `references/agent-loop.md` — full AgentSpec options, result fields, presets, doc URLs.
