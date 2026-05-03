# Domain: LiveView Architecture

- **Stateless components**: `live_component` modules should be stateless rendering helpers. State must live in the parent `LiveView`. Flag plans that propose managing meaningful state inside a `live_component`.
- **No data fetches inside components**: Components must not call context functions or run Ecto queries. Data fetching must happen in the parent LiveView (`mount/3`, `handle_params/3`, `handle_info/2`) and be passed as assigns.
- **Heavy work offloaded**: CPU-bound or I/O-bound operations inside LiveView event handlers should be offloaded to supervised `Task`s or `GenServer`s, with results sent back via `PubSub` or `send/2`.
- **PubSub lifecycle**: PubSub subscriptions must be set up in `mount/3` (guarded by `connected?/1`) and cleaned up in `terminate/2`. Flag plans that subscribe in `handle_params` or without `connected?/1` guard.
