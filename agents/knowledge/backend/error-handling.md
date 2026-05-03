# Domain: Error Handling

- **`{:ok, result}/{:error, reason}` tuples**: Use tagged tuples for all expected failure modes. Flag bare `nil` returns or raw `false` used to signal failure from public API functions.
- **`with` for failure sequences**: Multi-step operations where any step can fail must use `with`. Flag nested `case` statements that could be flattened with `with`.
- **No nested `with` blocks**: Nested `with` blocks hide the top-level flow and should never appear. Extract the inner sequence to a named `defp` function so the outer function has a single flat `with`. Flag any `with` block that contains another `with` inside its clauses or body.
- **No `try/rescue` for control flow**: `try/rescue` must only be used for truly exceptional, undefined states — not expected errors. Flag any `try`, `rescue`, or `catch` wrapping context calls, Ecto operations, or HTTP client calls. The bar is very high: if there is any other way to handle the failure, use it.
- **Raise for undefined state only**: `raise/1` and `throw/1` are for programmer errors and undefined states. Flag use of `raise` for user-facing errors (validation failures, not-found errors).
