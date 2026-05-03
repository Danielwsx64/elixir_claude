# Domain: OTP Code Patterns

- **`handle_call` return shape**: Must return `{:reply, result, new_state}` or `{:reply, result, new_state, :hibernate}`. Flag returns that omit state or use wrong tuple shape.
- **`handle_cast` return shape**: Must return `{:noreply, new_state}`. Flag returns that include a reply value.
- **`init/1` blocking work**: If `init/1` performs I/O, database queries, or calls external services, it must defer with `{:ok, state, {:continue, :init}}` and implement `handle_continue/2`. Flag `send(self(), :init)` anti-pattern — this leaves a window where messages arrive before state is ready.
- **`handle_info` catch-all**: GenServers that subscribe to PubSub or use `Process.monitor` must have a catch-all `handle_info/2` clause to prevent crashes on unexpected messages.
