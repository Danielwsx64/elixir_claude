# Domain: Documentation and Specs

- **`@moduledoc`**: Every module must have `@moduledoc "..."` or `@moduledoc false` (for intentionally private modules). Flag modules with no `@moduledoc`.
- **`@doc`**: Every public function in a context or public API module must have `@doc "..."`. Flag undocumented public functions.
- **`@spec`**: Every public function in a context or public API module must have a `@spec`. Flag missing specs.
- **`## Examples` with doctests**: Context functions should include `## Examples` sections with runnable doctests. Flag context functions with no examples.
