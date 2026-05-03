# Domain: Naming Conventions

- **Predicate functions**: Functions returning a boolean must end with `?` (e.g., `valid?/1`, `admin?/1`). Flag `is_valid`, `check_admin`, etc.
- **Raising variants**: Functions that raise on failure must end with `!` (e.g., `get_user!/1`). Flag raising functions without the `!` suffix.
- **Generic names**: Flag public API functions named `data`, `result`, `stuff`, `helper`, `utils`, `process`, `handle`, `do_thing`. Name by what the function does.
- **Abbreviations in public API**: Flag abbreviated parameter or function names in public modules (e.g., `usr`, `acct`, `msg`). Internal variables may use short names.
