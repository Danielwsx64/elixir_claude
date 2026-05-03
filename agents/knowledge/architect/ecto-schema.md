# Domain: Ecto Schema and Query Architecture

- **Correct association types**: Verify `has_many`/`belongs_to`/`has_one`/`many_to_many` match the intended cardinality and foreign key placement. Flag inverted or mismatched associations.
- **N+1 query detection**: Flag any plan that proposes iterating over a collection and calling a context function (which runs a query) inside the loop. The fix is to preload associations or use a batch query.
- **Polymorphic association justification**: Polymorphic associations (`belongs_to` with `:foreign_key` + `:references` gymnastics) are rarely idiomatic in Ecto. Flag proposals for polymorphic associations and ask whether a simpler schema design (e.g., separate join tables or union types) would serve better.
