# elixir-tooling

A Claude Code plugin providing expert Elixir/Phoenix development tooling for teams. Ships four agents that form a complete development workflow.

## Workflow

```
User creates plan
       │
       ▼
elixir-plan-reviewer   ← correctness gate (idioms, OTP, Ecto, Phoenix)
       │ ✅ APPROVED
       ▼
elixir-architect       ← structural gate (topology, context boundaries, security)
       │ ✅ ARCHITECTURE APPROVED
       ▼
elixir-qa              ← test contract (scenarios, ExUnit patterns, coverage checklist)
       │ scenarios defined
       ▼
  [implementation]
       │
       ▼
elixir-backend         ← code quality gate (style, naming, error handling, docs)
```

## Agents

### `elixir-plan-reviewer`

Reviews Claude Code implementation plans before you start coding. Checks:

- **Elixir idioms** — pattern matching, pipe operators, list access, predicate naming, `String.to_atom` safety, struct field access
- **OTP design** — GenServer signatures, supervision trees, process naming in child specs, `Task.async_stream` for concurrency
- **Ecto** — field types, `cast` security (no programmatic fields), association preloading, migration generation conventions
- **Phoenix** — context boundaries, router scope aliases, no `Phoenix.View`, LiveView layout (`<Layouts.app>`), flash and icon conventions
- **Module organisation** — no nested modules per file, proper context/domain grouping
- **Test strategy** — `start_supervised!/1`, no `Process.sleep`, `Process.monitor` + DOWN assertions, `:sys.get_state` for ordering
- **Plan completeness** — all referenced modules listed, verification steps present, migration naming correct

Output: **Critical / Important / Minor** issues → **✅ APPROVED** or **🔁 NEEDS REVISION**

---

### `elixir-architect`

Reviews structural and architectural decisions. Asks clarifying questions before rendering a verdict when the plan is ambiguous. Checks:

- **OTP process topology** — correct primitive (GenServer vs DynamicSupervisor+Registry vs Task), restart strategies, hierarchy circularity, atom process names from user input
- **Phoenix context boundaries** — clean public API modules, no cross-context internal access, no technical-layer naming (Services/Helpers/Utils)
- **LiveView architecture** — stateless components, no data fetches inside components, heavy work offloaded to supervised Tasks/GenServers, PubSub lifecycle
- **Ecto schema and query architecture** — correct association types, N+1 query detection, polymorphic association justification
- **Security architecture** — `String.to_atom` on user input, programmatic fields in `cast/3`, raw SQL string interpolation, auth checks at context layer

Output: Clarifying questions (if any) → **Critical / Important / Minor** issues → **✅ ARCHITECTURE APPROVED** or **🔁 ARCHITECTURE NEEDS REVISION**

---

### `elixir-qa`

Defines the complete test contract before any code is written. Produces:

- **ExUnit foundations** — `async: true` safety, test file path mirroring, setup strategy, `DataCase`/`ConnCase` selection
- **Context and business logic scenarios** — happy path, invalid changeset, constraint violations, auth boundary
- **OTP process tests** — `start_supervised!/1`, `:sys.get_state` assertions, crash/recovery, concurrent access; forbids `Process.sleep`
- **LiveView tests** — mount, event handlers (valid + invalid), redirect assertions, PubSub re-render, auth redirect
- **Property-based and edge cases** — `StreamData` for pure functions, boundary values, Unicode/empty strings
- **Integration and boundary tests** — strict hierarchy: real setup first (factories, args, test DB); else `Req.Test`/`Tesla.Mock` for HTTP; else `Mimic` (with `@spec` on mocked functions). `Mox` forbidden; flags anonymous-function mocks; email via `Swoosh.TestAssertions`

Output: Test file inventory table → per-module Given/When/Then scenarios with ExUnit patterns → coverage checklist

---

### `elixir-backend`

Reviews written code for quality and style. Reads actual source files. Checks:

- **Style and formatting** — 2-space indent, snake_case/CamelCase, no commented-out code, no `IEx.pry`/`IO.inspect` left in source
- **Naming conventions** — predicates end with `?`, raising variants end with `!`, no generic names, no abbreviations in public API
- **Pipe operators** — `|>` for chains of 2+, one step per line, `then/2` for branching, `tap/2` for side effects, no single-step pipes
- **Pattern matching and guards** — most-specific-first clause order, `^` pin usage, `%{}` ambiguity, exhaustive `case`
- **Error handling** — `{:ok}/{:error}` tuples, `with` for failure sequences, no `try/rescue` for control flow
- **Documentation and specs** — `@moduledoc`, `@doc`, `@spec`, `## Examples` with doctests on context functions
- **Ecto changesets** — no programmatic fields in `cast/3`, `get_field/2` not map access, named bindings, no `fragment` string interpolation
- **OTP code patterns** — `handle_call`/`handle_cast` return shapes, `{:continue, :init}` for blocking `init/1`, `handle_info` catch-all

Output: **Critical / Important / Minor** issues with file:line + current snippet + fix → "Positive Patterns Noted" → **✅ APPROVED** or **🔁 NEEDS REVISION**

---

## Installation

```bash
claude plugin install /path/to/elixir_tooling
```

To install at project scope (shared with collaborators via `.claude/settings.json`):
```bash
claude plugin install --scope project /path/to/elixir_tooling
```

After installation, reload plugins without restarting your session:
```
/reload-plugins
```

## Using the Agents in a Claude Session

### Automatic (recommended)

The agents trigger automatically based on context. Claude reads each agent's description
and delegates to the right agent at the right moment:

- After you share a plan → `elixir-plan-reviewer` runs
- After plan correctness is confirmed → `elixir-architect` reviews structure
- After architecture is approved → `elixir-qa` defines test scenarios
- After code is written → `elixir-backend` reviews quality

You don't need to invoke them explicitly — just work normally and Claude will
bring in the right specialist at each stage.

### Explicit invocation

You can also call any agent by name at any time:

```
Use the elixir-architect agent to review my context boundary design
Use the elixir-qa agent to define test scenarios for the Accounts module
Use the elixir-backend agent to review lib/my_app/billing/
```

### Browsing available agents

To see all installed agents in your session:
```
/agents
```

This shows all agents (built-in, user, project, and plugin), their descriptions,
and their configured tools.

## Sharing with coworkers

Point coworkers to this directory and have them run the install command above. No additional setup required — the plugin is self-contained.
