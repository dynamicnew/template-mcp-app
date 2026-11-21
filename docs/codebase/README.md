# Codebase architecture maps (optional, read on demand)

Add concise architecture maps here only when the codebase grows large enough
that agents would otherwise scan many files to understand how components fit
together. Do not duplicate code line by line; link to entrypoints and contracts.

## Suggested maps (create only as needed)

- `mcp-server-and-tools.md` — MCP server transport (stdio / Streamable HTTP),
  tool/resource registration, Zod schemas, auth, and `${PLUGIN_DATA}` storage.
- `ui-views-and-lifecycle.md` — `ui://` single-file views, `@modelcontextprotocol/ext-apps`
  lifecycle handlers, Antigravity entrypoints (`inline`, `aux_pane`, `global`,
  `file`), host styling, and CSP boundaries.
- `testing-and-packaging.md` — unit/protocol test harnesses, `basic-host` /
  Playwright setup, and Antigravity plugin bundling (`plugin.json`,
  `mcp_config.json`, `agy plugin validate`).
