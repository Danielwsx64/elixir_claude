# Domain: Phoenix Context Boundaries

- **Clean public API modules**: Each context must expose a single public API module (e.g., `MyApp.Accounts`). Internal modules (schemas, queries, etc.) should not be called directly across context boundaries.
- **No cross-context internal access**: Flag plans where one context's internal module (e.g., `MyApp.Accounts.UserQuery`) is called from another context. All cross-context calls must go through the public API.
- **No technical-layer naming**: Context names like `Services`, `Helpers`, `Utils`, or `Managers` are not domain concepts — flag them. Correct: name by business domain (e.g., `Billing`, `Notifications`, `Inventory`).
- **Umbrella app circular dependencies**: In umbrella applications, flag any proposed dependency where App A depends on App B and App B depends on App A.
- **Thin controllers**: Controllers must only extract params and format responses. All business logic — including loading resources by ID — belongs in the context module. Flag plans where controllers perform queries, contain conditional branching over data, or call internal data-layer modules directly.
- **Repo calls confined to data layer**: `Repo` calls must only appear in dedicated data-layer modules (Loaders, Mutators, or equivalent query/write modules). Context modules coordinate these but must not call `Repo` directly. Flag plans where context functions inline `Repo.get`, `Repo.get_by`, `Repo.insert`, etc. outside of dedicated data-layer modules.
