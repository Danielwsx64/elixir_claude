# Domain: OTP Process Topology

- **Correct primitive selection**: Verify each use case maps to the right OTP primitive:
  - Stateful singleton → `GenServer`
  - Dynamic pool of named, monitored processes → `DynamicSupervisor` + `Registry`
  - Concurrent enumeration over a collection → `Task.async_stream/3`
  - Fire-and-forget work with fault tolerance → supervised `Task` via `Task.Supervisor`
  - Flag plans that use a raw `GenServer` where a `DynamicSupervisor`+`Registry` is needed for dynamic, per-entity state.
- **Restart strategies**: `one_for_one` when workers are independent; `rest_for_one` when later workers depend on earlier ones; `one_for_all` only when all children share state and must restart together. Flag mismatches.
- **Supervision hierarchy circularity**: Flag plans where a supervised process attempts to start or name a supervisor that is also its own ancestor.
- **Atom-based process names from user input**: Flag any plan where a process name is derived from user-supplied data (atom exhaustion / DoS risk). Correct approach: use `Registry` with string or tuple keys.
