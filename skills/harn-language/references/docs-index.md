# Harn docs index — language & core reference

Fetch `.md` URLs (preferred over HTML). Harn is pre-1.0; docs track the current release.

## Start here
- Quick reference (LLM one-pager): https://harnlang.com/docs/llm/harn-quickref.md
- Scripting cheatsheet: https://harnlang.com/scripting-cheatsheet.md
- Language basics: https://harnlang.com/language-basics.md
- Best practices: https://harnlang.com/best-practices.md
- Cookbook: https://harnlang.com/cookbook.md
- Common tasks: https://harnlang.com/common-tasks.md
- Glossary: https://harnlang.com/concepts/glossary.md
- Mental model: https://harnlang.com/concepts/mental-model.md
- Choosing an agent abstraction: https://harnlang.com/concepts/abstraction-ladder.md

## Language
- Error handling: https://harnlang.com/error-handling.md
- Diagnostic codes (HARN-<CAT>-<NNN>): https://harnlang.com/diagnostics.md
- Reading shape diagnostics: https://harnlang.com/reading-shape-diagnostics.md
- Reuse narrowing checks: https://harnlang.com/narrowing-checks.md
- pick(): https://harnlang.com/pick.md
- Modules and imports: https://harnlang.com/modules.md
- Concurrency: https://harnlang.com/concurrency.md
- Streams: https://harnlang.com/streams.md
- Runtime context: https://harnlang.com/runtime-context.md
- Filesystem host capabilities: https://harnlang.com/host-capabilities/fs.md
- Builtins and Harness capabilities: https://harnlang.com/builtins.md
- Prompt templating (.harn.prompt): https://harnlang.com/prompt-templating.md
- Testing: https://harnlang.com/testing.md

## Specification
- Overview: https://harnlang.com/language-spec.md
- 01 Lexical rules: https://harnlang.com/spec/language/01-lexical-rules.md
- 02 Grammar (EBNF): https://harnlang.com/spec/language/02-grammar.md
- 03 Operator precedence: https://harnlang.com/spec/language/03-operator-precedence-table.md
- 04 Scope rules: https://harnlang.com/spec/language/04-scope-rules.md
- 05 Destructuring: https://harnlang.com/spec/language/05-destructuring-patterns.md
- 06 Evaluation order: https://harnlang.com/spec/language/06-evaluation-order.md
- 07 Runtime values: https://harnlang.com/spec/language/07-runtime-values.md
- 08 Binary operators (int overflow → float): https://harnlang.com/spec/language/08-binary-operator-semantics.md
- 09 Control flow: https://harnlang.com/spec/language/09-control-flow.md
- 10 Concurrency: https://harnlang.com/spec/language/10-concurrency.md
- 11 Pipeline lifecycle: https://harnlang.com/spec/language/11-pipeline-lifecycle.md
- 12 Error model: https://harnlang.com/spec/language/12-error-model.md
- 13 Functions and closures: https://harnlang.com/spec/language/13-functions-and-closures.md
- 14 Enums: https://harnlang.com/spec/language/14-enums.md
- 15 Structs: https://harnlang.com/spec/language/15-structs.md
- 16 Impl blocks: https://harnlang.com/spec/language/16-impl-blocks.md
- 17 Interfaces: https://harnlang.com/spec/language/17-interfaces.md
- 18 Attributes: https://harnlang.com/spec/language/18-attributes.md
- 19 Type annotations: https://harnlang.com/spec/language/19-type-annotations.md
- 20 Built-in methods: https://harnlang.com/spec/language/20-built-in-methods.md
- 21 Iterator protocol: https://harnlang.com/spec/language/21-iterator-protocol.md
- 22 Method-style builtins: https://harnlang.com/spec/language/22-method-style-builtins.md
- 23 Runtime errors: https://harnlang.com/spec/language/23-runtime-errors.md
- 24 OAuth: https://harnlang.com/spec/language/24-oauth.md
- 25 Persistent store: https://harnlang.com/spec/language/25-persistent-store.md
- 26 Checkpoint & resume: https://harnlang.com/spec/language/26-checkpoint-resume.md
- 27 Agent lifecycle: https://harnlang.com/spec/language/27-agent-lifecycle-suspend-resume.md
- 28 Host shell discovery: https://harnlang.com/spec/language/28-host-shell-discovery.md
- 29 Workspace manifest (harn.toml): https://harnlang.com/spec/language/29-workspace-manifest-harn-toml.md
- 30 Sandbox mode: https://harnlang.com/spec/language/30-sandbox-mode.md
- 31 Test framework: https://harnlang.com/spec/language/31-test-framework.md
- 32 Environment variables: https://harnlang.com/spec/language/32-environment-variables.md
- 33 Known limitations: https://harnlang.com/spec/language/33-known-limitations-and-future-work.md

## Stdlib (non-LLM)
- Postgres: https://harnlang.com/postgres.md — SQLite: https://harnlang.com/sqlite.md
- Cache: https://harnlang.com/stdlib/cache.md — Durable step: https://harnlang.com/stdlib/step.md
- Calendar: https://harnlang.com/stdlib/calendar.md — Timing: https://harnlang.com/stdlib/timing.md
- Edit (structured source edits): https://harnlang.com/stdlib/edit.md — Diff: https://harnlang.com/stdlib/diff.md
- GraphQL: https://harnlang.com/stdlib/graphql.md
- OAuth: https://harnlang.com/oauth.md — OAuth storage: https://harnlang.com/stdlib/oauth-storage.md
- Observability: https://harnlang.com/stdlib/observability.md
- Project scanning: https://harnlang.com/project-scan.md
- Secret store: https://harnlang.com/hostlib/secret_store.md
- Embeddings / similarity: https://harnlang.com/hostlib/embed.md

## Migrations (check when code looks outdated)
- const/let: https://harnlang.com/migrations/const-let.md
- Pure collection methods: https://harnlang.com/migrations/pure-collection-methods.md
- 0.10: https://harnlang.com/migrations/v0.10.md — 0.7: https://harnlang.com/migrations/v0.7.md
- Schema-as-type: https://harnlang.com/migrations/schema-as-type.md
- Template engine v2: https://harnlang.com/migrations/template-engine-v2.md

## Everything
- Index: https://harnlang.com/llms.txt — Full concatenated docs: https://harnlang.com/llms-full.txt
