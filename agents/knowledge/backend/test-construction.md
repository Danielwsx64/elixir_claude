# Domain: Test Construction

> **Scope**: These rules apply **only** to test files (`*_test.exs`, anything under `test/`). Never apply them to library code under `lib/`.

## Assertions & Verification

- **Every test must have at least one `assert` or `refute`.** Flag tests with no assertion.
- **Create/update tests — verify via `Repo.get_by`**: when a function returns a created/updated struct, use the returned value only to extract the `id`; assert the full committed state via `Repo.get_by` with all key/values.
  ```elixir
  # correct
  assert {:ok, rel} = Relationships.create(attrs)
  assert Repo.get_by(Relationship, id: rel.id, user_id: attrs.user_id, contact_id: attrs.contact_id)

  # wrong — asserting fields directly on the returned struct
  assert {:ok, rel} = Relationships.create(attrs)
  assert rel.user_id == attrs.user_id
  ```
  Exception: virtual fields (not persisted) must still be asserted directly on the returned struct.
- **Delete tests**: use `refute Repo.get_by(...)` to confirm the record is gone.
- **Changeset error tests**: use `errors_on(changeset)` (from `DataCase`) and compare the **full** map.
  ```elixir
  assert {:error, changeset} = Fn.call(%{})
  assert errors_on(changeset) == %{name: ["can't be blank"], owner_id: ["can't be blank"]}
  ```
- **Controller response assertions**:
  - Bind with `assert response = conn |> verb(...) |> json_response(status)` then assert the full map on a separate line.
  - Never do partial field assertions (`response["field"] == ...`).
  - Even empty 204 payloads must be asserted: `assert response == %{}`.
  - For responses with dynamic fields (e.g., a generated `id`): extract only the dynamic part first, then assert the full map.
  ```elixir
  assert response = conn |> post(~p"/api/x", params) |> json_response(201)
  assert %{"id" => id} = response
  assert response == %{"id" => id, "field" => "value"}
  ```

## Assert Style

- **Use `assert call() == expected`** when the full return value is statically known — no variable extraction needed.
- **Use pattern matching directly** (`assert pattern = call()`) only when you need to extract a dynamic value to use later.
- **Never combine both approaches** for the same assertion.
  ```elixir
  # wrong — two-step when == suffices
  result = Fn.init([])
  assert result == {:ok, %{initialized: false}}

  # wrong — two-step when pattern matching suffices
  result = Fn.handle(state)
  {:noreply, %{cached_at: cached_at}} = result
  assert result == {:noreply, %{initialized: true, cached_at: cached_at}}

  # correct — full value known
  assert Fn.init([]) == {:ok, %{initialized: false}}

  # correct — dynamic cached_at must be extracted
  assert {:noreply, %{initialized: true, cached_at: _}} = Fn.handle(state)
  ```

## Describe / Test Structure

- **One `describe` block per public function.** Never split scenarios for the same function into multiple `describe` blocks. Merge duplicates.
- **Test description labels**: short business scenario — no return values, no module/function namespaces in the label.
  ```elixir
  # correct
  test "returns error when email is missing"
  # wrong
  test "Accounts.create_user/2 returns {:error, changeset} when email missing"
  ```

## Mocking strategy

Strict hierarchy. Pick the highest level that applies — never skip levels.

### 1. No mock — real setup first

Before reaching for any mock, exhaust real setup options:

- Pass the dependency as a function argument (dependency injection).
- Use a factory to build the data the function expects.
- Hit the test DB with a real insert.
- Build a real struct or in-memory value.

Mock is the last resort, not the default.

```elixir
# correct — real factory + DB insert, no mock
user = insert(:user)
assert {:ok, profile} = Accounts.create_profile(user, valid_attrs())

# wrong — mocking what could be set up for real
expect(Repo, :get, fn _, _ -> %User{id: 1} end)
assert {:ok, profile} = Accounts.create_profile(1, valid_attrs())
```

### 2. HTTP-layer mock — `Req.Test` or `Tesla.Mock`

When the code under test makes HTTP calls, mock the **response** (status + body) at the transport layer. Never mock the calling module — that bypasses the real HTTP pipeline (decode, headers, retries).

- `Req.Test` for code that uses `Req`.
- `Tesla.Mock` for code that uses `Tesla`.
- Files using `Req.Test` or `Tesla.Mock` must use `async: false`.

```elixir
# correct — Req.Test intercepts the transport layer
expect(Client, fn conn -> json(conn, %{suggested_ids: [id]}) end)
assert {:ok, results} = Context.search(scope, "prompt")

# correct — Tesla.Mock intercepts at adapter level
Tesla.Mock.mock(fn
  %{method: :get, url: "https://api.example.com/users/1"} ->
    %Tesla.Env{status: 200, body: %{"id" => 1, "name" => "Alice"}}
end)
assert {:ok, %{name: "Alice"}} = Client.get_user(1)

# wrong — mocking the calling module bypasses HTTP entirely
expect(ML.Client, :suggest_contacts, fn _candidates, _prompt, _count -> {:ok, [id]} end)
```

#### Multiple sequential HTTP calls

Write one `expect` per HTTP call in the order the calls are made. Never use a single `stub` with `cond` routing.

```elixir
# correct — two ordered expects
expect(Client, fn conn -> json(conn, %{"token" => "abc"}) end)
expect(Client, fn conn -> text(conn, "sa@project.iam.gserviceaccount.com") end)

# wrong — single stub with cond routing
stub(Client, fn conn ->
  cond do
    String.ends_with?(conn.request_path, "/token") -> json(conn, ...)
    true -> text(conn, "sa@...")
  end
end)
```

#### `Req.Test` import rule

- If the file does **not** use `use Mimic`: `import Req.Test` at the top; use bare `expect/json/text`.
- If the file **also** uses `use Mimic`: `alias Req.Test` (no import — `expect` clashes with Mimic); use `Test.expect/Test.json/Test.text`.
- Never write `Req.Test.` fully qualified inside code bodies.

#### `Req.Test` response helpers

Use `json(conn, body)` or `text(conn, body)` — never `Plug.Conn.send_resp` or `Plug.Conn.put_status` directly. For non-200 responses: `%{conn | status: 422} |> text("")`.

### 3. Mimic — only when (1) and (2) don't apply

Use Mimic only for internal collaborators that don't go through HTTP and whose real setup is genuinely infeasible (e.g., third-party SDK with global side-effects).

- **Mocked functions must have a `@spec`** on the target module. Without a typed contract the test mocks a fiction — the real function signature can drift and the test won't catch it. Treat a missing `@spec` as a blocker.
- **Never put `expect`/`stub` inside `setup`** — inline in each test.
- **Prefer `expect` over `stub`** — only use `stub` when the call count is variable or the call order is non-deterministic.

### Forbidden

- **`Mox` is not accepted in this project.** Replace with one of the three strategies above.
- Anonymous-function mocks passed as arguments (e.g., `fn _ -> :ok end`) — they hide the contract.
- Any other mock library or pattern.

### Exception: email

Never mock email notifiers. Use `assert_email_sent` (from `Swoosh.TestAssertions`) — it's an in-memory test adapter, not a mock.

## Aliases & Naming

- **Alias all project modules** so the project's root namespace (e.g., `MyApp.*`) is never spelled out inside test bodies. Add the alias at the top of the module, alphabetically.
- **Single alias per line, alphabetical order** — no multi-alias syntax (`alias Mod.{A, B}`).
  ```elixir
  # correct
  alias MyApp.Auth.User
  alias MyApp.Relationships

  # wrong
  alias MyApp.Auth.{User, UserToken}
  ```
- **No abbreviated variable names** — use full descriptive names everywhere. Exception: single-letter bindings in Ecto query macros (`from u in User, where: u.id == ^id`) are allowed.

## General

- **No `rescue`, `try`, or `catch`** unless absolutely unavoidable. Flag any such clauses.
