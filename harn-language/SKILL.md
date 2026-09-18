---
name: harn-language
description: Write, read, or fix Harn (.harn) source code — syntax, types, control flow, modules, errors, concurrency, streams. Use whenever a task touches .harn files or harn.toml or mentions the Harn language (harnlang.com). Not for LLM/agent APIs specifically (see harn-llm-agents) or triggers/connectors (see harn-triggers-orchestration).
---

# Harn language

Harn is a pipeline-oriented language and runtime for orchestrating AI agents
(https://harnlang.com). It is **pre-1.0**: APIs change between releases. When
unsure, fetch the markdown page for the topic (`https://harnlang.com/<page>.md`,
see `references/docs-index.md`) instead of guessing, and run `harn check`.

## Non-negotiable rules

1. **Entry points take the Harness.** `fn main(harness: Harness) { ... }` or
   `pipeline default(harness: Harness) { ... }`. All effects (fs, stdio, llm,
   clock, process, env, runtime) go through `harness.*`.
2. **Effect authority: pass the narrowest handle.** Keep root `Harness` at
   entrypoints/coordinators; helpers take a sub-handle:
   ```harn
   fn load_config(fs: HarnessFs, path: string) -> string {
     return fs.read_text(path)
   }
   fn main(harness: Harness) {
     harness.stdio.println(load_config(harness.fs, "harn.toml"))
   }
   ```
3. **`const` by default, `let` only when the binding is reassigned or mutated.**
   Collection methods are pure: `items = items.appending("a")` (calling
   `base.appending("a")` alone is a no-op, lint HARN-LNT-066).
4. **`match` must be exhaustive** — a missing variant is a hard error.
5. **Multiline strings use `"""..."""`**, not heredocs (`<<TAG` is only valid
   inside LLM tool-call JSON). Interpolation: `"Hello, ${name}!"`. Raw: `r"C:\x"`.
6. **Strings are UTF-8; indexing is O(n).** For char scans, `const cs = chars(src)`
   once, then index the list.
7. **Always validate:** `harn fmt <path>` → `harn check <path>` → `harn lint <path>`
   → `harn test`. Use `harn explain HARN-XXX-NNN` for diagnostic codes.

## Syntax cheat sheet

```harn
// bindings & types (gradual typing: omit, `unknown` at trust boundaries, `any` = escape hatch)
const x: int = 42
let n = 0
const nums: list<int> = [1, 2, 3]
const cfg: {host: string, port?: int} = {host: "localhost"}
type Config = {model: string, max_tokens: int}
type Verdict = "pass" | "fail" | "unclear"
const row: tuple<string, int> = ["retries", 3]
const price = decimal("19.99")
const d = 500ms   // also 5s, 2m, 1h

// functions & closures
fn add(a: int, b: int) -> int { return a + b }
fn sum(...nums) { return nums.reduce(0, { acc, x -> acc + x }) }
const square = { x -> x * x }
pub fn greet(name: string) -> string { return "Hello, ${name}" }   // pub = exported

// control flow — `if` and `try/catch` are expressions
const grade = if score >= 90 { "A" } else if score >= 80 { "B" } else { "C" }
guard x > 0 else { return "invalid" }
require len(name) > 0, "name cannot be empty"
match status {
  "active" -> { harness.stdio.log("Running") }
  "stopped" -> { harness.stdio.log("Halted") }
}
for {index, value} in items.enumerate() { }
for [a, b] in xs.zip(ys) { }
for {key, value} in dict.entries() { }       // sorted by key
for i in 1 to 5 { }             // inclusive
for i in 0 to 3 exclusive { }   // half-open
for i in range(3, 7) { }
while i < 10 { i = i + 1 }

// collections
xs.map({ x -> x * 2 }).filter({ x -> x > 3 })
xs.iter().filter(...).map(...).take(3).to_list()   // lazy
xs[1:4]  s[-5:]  "a" in xs  "k" not in dict  user?.name ?? "anon"
const {name, role = "user"} = person
const [first, ...rest] = items
data |> { l -> l.filter({ x -> x > 0 }) } |> json_stringify
"hello world" |> split(_, " ")
const sub = pick(record, ["a", "c"])

// structs, impl, enums, interfaces (Go-style implicit satisfaction)
struct Point {
  x: int
  y: int
}
impl Point {
  fn norm(self) { return sqrt(self.x * self.x + self.y * self.y) }
}
const p = Point { x: 3, y: 4 }
enum Status {
  Active
  Pending(reason)
  Failed(code, message)
}
const s = Status.Pending("waiting")
match s.variant {
  "Active" -> { harness.stdio.log("ok") }
  "Pending" -> { harness.stdio.log(s.fields[0]) }
  "Failed" -> { harness.stdio.log(s.fields[1]) }
}
interface Displayable { fn display(self) -> string }
fn log_item<T>(item: T) where T: Displayable { }

// discriminated unions narrow automatically
type Msg = {kind: "ping", ttl: int} | {kind: "pong", latency_ms: int}

// narrowing unknown
fn handle(v: unknown) -> string {
  if type_of(v) == "string" { return v.upper() }
  if schema_is(v, MyShape) { return v.name }
  return "other"
}
fn is_text(v: unknown) -> v is string { return type_of(v) == "string" }

// attributes
@deprecated(since: "0.8", use: "compute_v2")
@test
pub fn compute(x: int) -> int { return x + 1 }
```

## Errors and Results

```harn
const r = try { json_parse(raw) }                  // Result.Ok / Result.Err
const text = r?.text ?? "fallback"                  // ?. short-circuits on Err
const cfg = try { json_parse(raw) } catch (e) { default_config() }
fn compute(x) {
  const half = divide(x, 2)?    // postfix ? unwraps Ok or returns the Err early
  return Ok(half + 10)
}
const resp = try* harness.llm.call(prompt)          // rethrow into enclosing catch
const out = try { fetch(p) } catch (e: ApiError) { fallback(e) }   // typed catch
throw "message"
retry 3 { risky() }                                  // retry block
```

## Concurrency and streams

```harn
const h = spawn { long_work() }
const v = await(h)
const doubled = parallel each xs { x -> x * 2 }                    // fail-fast
const out = parallel settle paths with { max_concurrent: 4 } { p -> grade(p) }
harness.stdio.log(out.succeeded, out.failed)                       // out.results: Ok/Err per item
defer { cleanup() }

gen fn numbers() -> Stream<int> {
  emit 1
  emit 2
}
for n in numbers() { }
stream.collect(stream.take(ch, 3), {max: 3})   // always bound collect with {max: N}
```
Shared mutable state across functions: `harness.runtime.atomic(0)`,
`atomic_add`, `atomic_get` (module-level `let` mutation is not fully supported).

## Modules

```harn
import "lib/helpers"                         // relative to current file, .harn optional
import { unified_diff } from "std/diff"      // stdlib uses std/ prefix
import "std/math"
```
Most builtins (`harness.llm.call`, `agent_loop`, `parallel`, `json_parse`,
`regex_*`, `jq`, `transcript_*`, `mcp_*`) are global — no import needed.
Top-level `const`/`fn` are visible to functions in the same file.

## Useful builtins

`json_parse`, `json_stringify`, `json_pointer(v, "/a/0")`, `jq(v, ".users[]")`,
`jq_first`, `schema_report`, `regex_match/replace/captures/split`, `format("{name}", {...})`,
`split`, `join`, `substring(s, start, end)`, `chars`, `set(...)`, `set_union`,
`group_by`, `partition`, `sha256`, `harness.fs.read_text/exists/render_template/render_prompt`,
`harness.env.get/get_or`, `harness.clock.now_ms/sleep_ms`, `harness.stdio.println/log/read_line`,
`argv` (list<string> of positional CLI args after `--`).

## References

- `references/docs-index.md` — every language/reference page URL, grouped.
- Canonical one-pager: https://harnlang.com/docs/llm/harn-quickref.md
- Spec: https://harnlang.com/language-spec.md
