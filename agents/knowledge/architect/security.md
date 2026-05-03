# Domain: Security Architecture

- **`String.to_atom/1` on user input**: Flag any plan that proposes deriving process names, map keys, or module names from user-supplied data via `String.to_atom/1`. Correct: use `Registry` with string/tuple keys, or `String.to_existing_atom/1` with a known safe set.
- **Programmatic fields in `cast/3`**: Fields set by the system (e.g., `user_id`, `role`, `inserted_by`) must not appear in `cast/3`. Flag plans that allow these through the changeset from user params.
- **Raw SQL string interpolation**: Flag any plan proposing `Repo.query("SELECT ... #{user_input}")` or `fragment("... #{var}")`. Correct: parameterised queries or `fragment/1` with `?` placeholders.
- **Auth checks at context layer**: Authorisation logic must be enforced inside context functions, not only at the controller/LiveView layer. Flag plans where auth is only checked at the boundary and context functions are called without access checks.
