# agent_loop reference (from https://harnlang.com/llm/agent_loop.md)

Pre-1.0 — re-check the live page if an option is rejected by `harn check`.

## How it works
1. Send prompt to model; read response.
2. If `loop_until_done: true`: native-tool mode treats final text with no tool
   calls as completion; text/no-tool mode looks for the done sentinel
   (`##DONE##`). If not complete, send a nudge and repeat until done or limits.
3. If `loop_until_done: false` (default): return after the first response.

## AgentSpec options (selected)
| Key | Default | Meaning |
|---|---|---|
| `profile` | `"tool_using"` | `tool_using` / `researcher` / `verifier` / `completer` |
| `provider`, `model` | — | route |
| `tools` | nil | ToolRegistry |
| `tool_format` | — | `"native"`, `"json"`, text |
| `history` | nil | caller-managed messages |
| `loop_until_done` | false | keep looping until completion |
| `done_sentinel` | mode-aware | `"##DONE##"` for text/no-tool, nil for native |
| `output` | nil | `"text"`, `"json"`, schema dict — terminal-answer contract |
| `max_iterations` | 50 | LLM round-trip cap |
| `iteration_budget` | nil | `{mode: "adaptive", initial, max, extend_by}` or fixed |
| `loop_control` | nil | `state -> nil | {action: "extend", by, reason}` |
| `max_nudges` | 8 | consecutive text-only replies allowed |
| `llm_caller` | nil | middleware-wrapped caller |
| `on_delta` | nil | streaming callback |
| `reasoning_policy` | `"auto"` | `auto`, `off`, or level |
| `tool_retries` / `tool_backoff_ms` | 0 / 1000 | tool retry policy |
| `max_concurrent_tools` | 1 | in-flight tool calls per phase |
| `tool_surface_narrowing` | enabled, window 5 | drop unused tools between turns |
| `policy` | nil | capability ceiling |
| `daemon`, `persist_path`, `resume_path`, `wake_interval_ms`, `watch_paths` | — | daemon mode |
| `compaction` | `{strategy: "hybrid", keep_last_n: 10}` | context-window policy |
| `compact_threshold`, `auto_compact`, `compact_callback` | — | compaction tuning |
| `transcript_projection`, `context_callback`, `message_decorator` | — | per-turn context shaping |
| `scratchpad` | false | session-local working memory |
| `deadline_ms` | nil | wall-clock limit |
| `verify_completion` | nil | hook when about to stop |
| `verify_completion_judge`, `turn_end_condition`, `step_judge` | nil | structured judges |
| `input_guardrail` | nil | pre-loop guardrail |
| `stop_after_successful_tools` | nil | stop once these tools succeed |
| `require_successful_tools` | nil | fail unless these succeed (inner list = any-of) |
| `require_artifacts` | nil | paths the run must produce |
| `stall_diagnostics` | nil | detect repeated tool calls |
| `skills`, `skill_match`, `working_files` | — | Harn skill activation |
| `mcp_servers` | nil | MCP servers for this loop |
| `llm_transcript_dir` | nil | write llm_transcript.jsonl |

## Result fields
`status` (`done`, `completion_unverified`, `error`, `input_guardrail`,
`suspended`, `stuck`, `budget_exhausted`, `provider_error`, `idle`, `watchdog`,
`failed`), `error`, `terminal`, `text`, `visible_text`, `output`,
`output_valid`, `llm.{iterations,duration_ms,input_tokens,output_tokens}`,
`tools.{calls,successful,rejected,mode}`, `transcript`, `trace`,
`adaptive_budget`, `stall_warnings`, `suspected_loop`, `handle` (when suspended).

## Presets (`std/agent/presets`)
| Preset | Use | Budget |
|---|---|---|
| audit | read-only inspection (verifier) | adaptive 4→12 |
| repair | tool-using fix | adaptive 4→16 |
| summary | one-shot, `tool_choice: "none"` | fixed 1 |
| verify | verifier with turn_end_condition | adaptive 1→5 |
| merge_captain / review_captain / oncall_captain / release_captain | long-running ops personas | adaptive, stall diagnostics on |

Register your own: `agent_preset_register("triage", {family: "captain", pack: {...}})`.

## Doc URLs
- Overview: https://harnlang.com/llm-and-agents.md
- LLM calls: https://harnlang.com/llm/llm_call.md
- Agent loops: https://harnlang.com/llm/agent_loop.md
- Completion control: https://harnlang.com/llm/completion-control.md
- Tools: https://harnlang.com/llm/tools.md
- Tool middleware: https://harnlang.com/stdlib/tool-middleware.md
- LLM caller middleware: https://harnlang.com/stdlib/llm-handlers.md
- Streaming & transcripts: https://harnlang.com/llm/streaming.md
- Providers: https://harnlang.com/llm/providers.md — setup: https://harnlang.com/provider-setup.md
- Provider matrix: https://harnlang.com/provider-matrix.md
- Ensembles: https://harnlang.com/llm/ensemble.md — Rerank: https://harnlang.com/llm/rerank.md
- Long-running tools: https://harnlang.com/long-running-tools.md
- Sessions: https://harnlang.com/sessions.md — Agent state: https://harnlang.com/agent-state.md
- Memory: https://harnlang.com/memory.md
- Agent lifecycle (suspend/resume): https://harnlang.com/agent-lifecycle.md
- Daemons: https://harnlang.com/stdlib/daemon.md
- Guardrails: https://harnlang.com/stdlib/agent-guardrails.md — Completion gate: https://harnlang.com/stdlib/agent-judge.md
- Governors: https://harnlang.com/stdlib/governors.md
- Skills (Harn-native): https://harnlang.com/skills.md
- Personas: https://harnlang.com/personas.md
- Human in the loop: https://harnlang.com/hitl.md
- Prompt assembly: https://harnlang.com/prompt-assembly.md — System reminders: https://harnlang.com/system-reminders.md
- Workflow runtime: https://harnlang.com/workflow-runtime.md
- MCP & ACP: https://harnlang.com/mcp-and-acp.md
- Tutorials: https://harnlang.com/tutorials/build-your-first-workflow.md, https://harnlang.com/tutorial-code-review-agent.md, https://harnlang.com/tutorial-eval-pipeline.md, https://harnlang.com/tutorial-daemon-agent.md, https://harnlang.com/tutorial-mcp-server.md
