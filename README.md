# Harn skills for OpenAI Codex

Codex skills for [Harnlang](https://harnlang.com), the pipeline-oriented language
and runtime for orchestrating AI agents. They were built from
[harnlang.com/llms.txt](https://harnlang.com/llms.txt).

| Skill | Use it for |
|---|---|
| `harn-language` | Writing and fixing `.harn` code: syntax, types, modules, errors, concurrency, streams |
| `harn-llm-agents` | `harness.llm.call`, structured output, tools, `agent_loop`, sub-agents, providers |
| `harn-triggers-orchestration` | `[[triggers]]` in `harn.toml`, connectors, cron/webhooks, `harn orchestrator serve` |
| `harn-cli-workflow` | `harn init/new`, `fmt/check/lint/test`, `run`, debugging, replay, deploy |

## Install

Codex finds skills in `.agents/skills/` at the project root, or in
`~/.agents/skills/` for every project. Clone this repo as `.agents`:

```bash
# one project (as a submodule)
git submodule add <repo-url> .agents

# all projects
git clone <repo-url> ~/.agents
```

Restart Codex after installing so it picks up the skills.

## Use

Codex loads a skill on its own when a task matches the skill's description.
To call one directly, name it in your prompt, for example
`$harn-llm-agents build an agent that triages GitHub issues`, or pick it
from `/skills`.

## Layout

```
skills/<name>/
  SKILL.md             instructions + frontmatter (name, description)
  agents/openai.yaml   Codex UI metadata and invocation policy
  references/          longer docs, loaded only when needed
```

## Caveat

Harn is pre-1.0, so its APIs change between releases. The skills tell Codex
to check the live docs (`https://harnlang.com/<page>.md`) and run
`harn check` when unsure. If a skill and the docs disagree, trust the docs,
and update the skill.
