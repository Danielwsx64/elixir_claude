# Domain: Pattern Matching and Guards

- **Clause order**: Most specific clauses must come before general ones. Flag function heads where a catch-all clause appears before a specific pattern.
- **Pin operator**: `^var` must be used to match against an existing variable's value. Flag cases where a variable is rebound inside a match when pinning was intended.
- **`%{}` as "match any map"**: Using `%{}` in a function head matches any map — this is usually a bug when `is_map/1` guard or a more specific pattern was intended. Flag ambiguous `%{}` catch-alls.
- **Exhaustive `case`**: Every `case` must either be exhaustive (all branches covered) or intentionally allow `CaseClauseError` to propagate. Flag `case` blocks where a common path is silently unhandled.
