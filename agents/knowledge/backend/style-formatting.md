# Domain: Style and Formatting

- **Indentation**: 2-space indent throughout. Flag tabs or 4-space indentation.
- **Naming case**: Module names `CamelCase`, functions and variables `snake_case`. Flag violations.
- **Commented-out code**: Flag any `# old code` or `# TODO: remove` blocks left in source. Dead code must be deleted, not commented out.
- **Debug artifacts**: Flag any `IEx.pry`, `IO.inspect`, `dbg/1`, or `require IEx` left in non-test source files.
