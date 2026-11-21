# Read-list: review

Use for checking a diff or a plan.

- **Read:** the diff or files under review, the tests that cover them, and the
  matching rule (`mcp-app-protocol-and-ui`, `copilot-host-integration`,
  `antigravity-host-integration`, `mcp-server-and-transports`, or
  `testing-and-verification`) or decision they must follow.
- **Do not read:** the whole handoff or unrelated modules.
- **Do not:** edit files or install/mutate global `~/.gemini/` or `~/.copilot/` state.
- **Check:** `@modelcontextprotocol/ext-apps` v2 + split MCP SDK v2 deps,
  handlers before `app.connect()`, `_meta.ui.csp` inside `contents[]`, Copilot
  tool disambiguation & `readOnlyHint`/editable-arg validation, Antigravity
  separate `file` tools & `-32601` fallback & eager persistence, `<= 10 MiB`
  HTML / `<= 1 MiB` tool result limits, plain-text CLI fallback, `${PLUGIN_ROOT}`
  / `${PLUGIN_DATA}` & secret hygiene, and honest host evidence separation.
- **Return:** only findings you are confident about, each with file (line
  numbers are fine in ephemeral chat replies), problem, impact, and a suggested
  fix. Say plainly if there are none and what you could not verify.
