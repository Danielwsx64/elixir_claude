# Domain: Pipe Operators

- **Use `|>` for chains of 2+**: A chain of two or more data transformations must use `|>`. Flag intermediate variable assignments that obscure the pipeline.
- **One step per line**: Each `|>` step must be on its own line. Flag inline chaining like `a |> b |> c` on one line when there are 3+ steps.
- **`then/2` for branching**: Use `then/2` when the pipeline needs to branch based on the accumulated value. Flag `case result do` immediately after a long pipeline — refactor with `then/2`.
- **`tap/2` for side effects**: Use `tap/2` for logging, metrics, or side-effect calls within a pipeline. Flag pipelines that assign to `_` just to trigger a side effect.
- **No single-step pipes**: `value |> some_function()` with no preceding or following steps — remove the pipe, call the function directly.
