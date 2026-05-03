# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A Claude Code plugin that provides expert Elixir/Phoenix development tooling. It ships four agents that form a complete development workflow — from plan review through architecture, test contract definition, and code quality review.

This project contains no Elixir code, no build system, and no dependencies. It is a plugin distribution package only.

## Project structure

- `agents/elixir-plan-reviewer.md` — Plan correctness review (idioms, OTP signatures, Ecto/Phoenix conventions)
- `agents/elixir-architect.md` — Structural and architectural review (process topology, context boundaries, security)
- `agents/elixir-qa.md` — Test contract definition (scenarios, ExUnit patterns, coverage checklist)
- `agents/elixir-backend.md` — Code quality review (style, naming, pipes, error handling, docs, OTP patterns)
- `.claude-plugin/plugin.json` — Plugin manifest (name, description, author)
- `.claude-plugin/marketplace.json` — Marketplace schema for Claude plugin registry

## Agent definition format (`agents/*.md`)

Agent files use YAML frontmatter followed by a Markdown system prompt:

```
---
name: <agent-id>
description: <used by Claude to decide when to trigger proactively; supports \n-escaped newlines and <example> blocks>
model: opus | sonnet | haiku
color: <ui color>
tools: Comma, Separated, Tool, Names
---

<system prompt in Markdown>
```

The `description` field is what Claude reads to decide whether to proactively invoke the agent. Keep examples in the description concrete and contextual.

## Installation

```bash
claude plugin install /path/to/elixir_tooling
```

## Workflow sequence

The four agents form a sequential handoff chain:

```
User creates plan
       │
       ▼
elixir-plan-reviewer   ← correctness gate (idioms, OTP signatures, Ecto/Phoenix)
       │ APPROVED
       ▼
elixir-architect       ← structural gate (process topology, context boundaries, security)
       │ ARCHITECTURE APPROVED
       ▼
elixir-qa              ← test contract (scenarios, ExUnit patterns, coverage checklist)
       │ (scenarios defined)
       ▼
  [implementation]
       │
       ▼
elixir-backend         ← code quality gate (style, naming, pipes, error handling, docs)
```

### Handoff conditions

| From | Condition | To |
|---|---|---|
| — | Plan exists in conversation | `elixir-plan-reviewer` |
| `elixir-plan-reviewer` | ✅ APPROVED verdict | `elixir-architect` |
| `elixir-architect` | ✅ ARCHITECTURE APPROVED verdict | `elixir-qa` |
| `elixir-qa` | Test scenarios defined | Implementation begins |
| Implementation | Code written or modified | `elixir-backend` |

## Agent domain summaries

### `elixir-plan-reviewer` (cyan, opus)
Reviews plans for low-level correctness:
1. **Elixir Idioms** — list access, pipe operators, predicate naming (`?` not `is_`), `String.to_atom` safety, struct dot-access, immutability
2. **OTP Design** — GenServer signatures, supervision trees, child spec `name:` keys, `Task.async_stream` for concurrency
3. **Ecto** — `:string` not `:text` field types, `cast` security (no programmatic fields), preloading, migration via `mix ecto.gen.migration`
4. **Phoenix** — context boundaries, no redundant `alias` in router scopes, no `Phoenix.View`, LiveView layout with `<Layouts.app>`, flash/icon conventions
5. **Module Organisation** — no multiple top-level modules per file, context/domain grouping
6. **Test Strategy** — `start_supervised!/1`, no `Process.sleep`, `Process.monitor` + DOWN assertions, `:sys.get_state` for ordering
7. **Plan Completeness** — all modules listed in files section, verification steps, migration naming, `mix deps.get` for new deps

### `elixir-architect` (orange, opus)
Reviews structural and architectural decisions:
1. **OTP Process Topology** — correct primitive, restart strategies, no hierarchy circularity, no atom names from user input
2. **Phoenix Context Boundaries** — clean public API, no cross-context internal access, no technical-layer naming
3. **LiveView Architecture** — stateless components, no data fetches in components, offload heavy work, PubSub lifecycle
4. **Ecto Schema and Query Architecture** — correct associations, N+1 detection, polymorphic association justification
5. **Security Architecture** — `String.to_atom` on user input, programmatic fields in `cast`, raw SQL interpolation, auth at context layer

### `elixir-qa` (green, opus)
Defines the complete test contract before implementation:
1. **ExUnit Foundations** — `async: true` safety, test file path mirroring, setup strategy, Case module selection
2. **Context and Business Logic** — happy path, invalid changeset, constraint violations, auth boundary
3. **OTP Process Tests** — `start_supervised!/1`, `:sys.get_state`, crash/recovery, concurrent access; no `Process.sleep`
4. **LiveView Tests** — mount, event handlers, redirect assertions, PubSub re-render, auth redirect
5. **Property-Based and Edge Cases** — `StreamData` for pure functions, boundary values, Unicode/empty strings
6. **Integration and Boundary Tests** — `Mox.expect/3`, `Mox.stub_with/2`, flag ad-hoc anonymous function mocks

### `elixir-backend` (purple, sonnet)
Reviews written code for quality:
1. **Style and Formatting** — 2-space indent, snake_case/CamelCase, no commented-out code, no debug artifacts
2. **Naming Conventions** — predicates end with `?`, raising variants end with `!`, no generic names, no abbreviations
3. **Pipe Operators** — `|>` for chains of 2+, one step per line, `then/2` for branching, `tap/2` for side effects
4. **Pattern Matching and Guards** — clause order, pin operator, `%{}` ambiguity, exhaustive `case`
5. **Error Handling** — tagged tuples, `with` for sequences, no `try/rescue` for control flow, raise for undefined state only
6. **Documentation and Specs** — `@moduledoc`, `@doc`, `@spec`, `## Examples` with doctests for context functions
7. **Ecto Changesets** — no programmatic fields in `cast`, `validate_required` after `cast`, `get_field/2`, named bindings, no `fragment` interpolation
8. **OTP Code Patterns** — `handle_call`/`handle_cast` return shapes, `{:continue, :init}` for blocking `init/1`, `handle_info` catch-all

Output format: **Critical / Important / Minor** issues grouped by severity, ending with **✅ APPROVED** or **🔁 NEEDS REVISION**.
