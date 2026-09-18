---
name: harn-cli-workflow
description: Use the harn CLI to scaffold, validate, run, test, debug and deploy Harn projects — harn init/new, fmt/check/lint/fix, run, test, playground, replay, portal, doctor, models/provider setup, --json mode. Use when setting up a Harn project, verifying Harn changes, or diagnosing a failing Harn run.
---

# Harn CLI workflow

## Setup
```bash
harn --version
harn init my-project                       # harn.toml + starter files
harn new my-agent --template agent         # templates: basic, agent, chat, mcp-server, eval, pipeline-lab, package, connector
harn doctor --check-providers              # readiness check
harn quickstart --non-interactive --provider ollama
harn models recommend | harn models list [--provider p] | harn models test <model> --provider <p>
```
Set provider keys via env (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, …) or
`harn-secret://…` references. Use `provider: "mock"` for credential-free runs.

## The verify loop (run after every change)
```bash
harn fmt src/                 # or --check in CI
harn check src/ --strict      # type-check + preflight, no execution
harn lint src/ [--fix]
harn fix --plan main.harn     # machine-applicable repairs; --apply --safety behavior-preserving
harn test [--filter name]     # user tests (@test fns) / conformance
harn explain HARN-TYP-014     # explain any diagnostic code
```
Every command supports `--json`, returning
`{ "schemaVersion": N, "ok": bool, "data": ..., "error": ..., "warnings": [] }`
— prefer it when parsing output programmatically.

## Running
```bash
harn run main.harn -- arg1 arg2           # args → `argv` inside the script
harn run -e 'harness.stdio.log("hi")'     # inline
harn run --trace main.harn                # also --profile, --deny/--allow, --sandbox-write-root
harn playground --script pipeline.harn --task "Explain this repo" --llm-mock --watch
harn repl
harn chat <model>
harn viz main.harn --output graph.mmd     # Mermaid flowchart
```
Scripts can start with `#!/usr/bin/env harn`.

## Debugging and determinism
- `harn test-bench run script.harn --llm-record fixture.jsonl` then `harn test-bench replay` — hermetic runs with mock clock.
- `harn test --record` / `--replay` for LLM fixtures.
- `harn session list --json`, `harn session export` — inspect persisted sessions.
- Replay/time-travel: https://harnlang.com/cookbooks/replay-time-travel.md
- `harn portal` — local observability UI; `harn usage` — LLM spend rollups.
- `harn bench main.harn --iterations 25` / `harn time run main.harn`.

## Editor
`harn-lsp` (LSP) + DAP — https://harnlang.com/editor-setup.md

## Deploy
Render / Fly.io / Railway templates under `deploy/`; long-running services use
`harn orchestrator serve` (see harn-triggers-orchestration).

## Doc URLs
- CLI reference: https://harnlang.com/cli-reference.md
- --json contract: https://harnlang.com/cli-json-contract.md
- Getting started: https://harnlang.com/getting-started.md
- Provider setup: https://harnlang.com/provider-setup.md
- Debugging: https://harnlang.com/debugging.md — Playground: https://harnlang.com/playground.md
- Testing: https://harnlang.com/testing.md — Diagnostics: https://harnlang.com/diagnostics.md
- Configuration: https://harnlang.com/configuration.md — harn.toml: https://harnlang.com/spec/language/29-workspace-manifest-harn-toml.md
- Sandboxing: https://harnlang.com/sandboxing.md
- Extending the CLI in Harn: https://harnlang.com/cli-extending-in-harn.md
