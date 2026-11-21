# Read-list: review

Use for checking a diff or a plan.

- **Read:** the diff or files under review, the tests that cover them, and the
  matching rule (`mcp-app-protocol-and-ui`, `antigravity-ui-entrypoints`,
  `mcp-server-and-packaging`, or `testing-and-verification`) or decision they
  must follow.
- **Do not read:** the whole handoff or unrelated modules.
- **Do not:** edit files or install/mutate global plugin state.
- **Check:** `@modelcontextprotocol/ext-apps` v2 + split MCP SDK v2 deps,
  handlers registered before `app.connect()`, `_meta.ui.csp` inside
  `contents[]`, separate `file` entrypoint tools, eager persistence (no teardown
  notification), `<= 10 MiB` HTML / `<= 1 MiB` tool result limits, plain-text
  CLI fallback, `${PLUGIN_ROOT}`/`${PLUGIN_DATA}` & secret hygiene, and honest
  separation of unit/`basic-host` passes vs live Antigravity App checks.
- **Return:** only findings you are confident about, each with file (line
  numbers are fine in ephemeral chat replies), problem, impact, and a suggested
  fix. Say plainly if there are none and what you could not verify.
