# Codebase architecture maps (optional, read on demand)

Add concise architecture maps here only when the codebase grows large enough
that agents would otherwise scan many files to understand how components fit
together. Do not duplicate code line by line; link to entrypoints and contracts.

## Suggested maps (create only as needed)

- `mcp-server-and-tools.md` — stdio / Streamable HTTP transport, split data vs.
  render tools, Zod schemas, annotations, auth, and `${PLUGIN_DATA}` / server
  storage.
- `ui-views-and-hosts.md` — `ui://` single-file views,
  `@modelcontextprotocol/ext-apps` lifecycle handlers, GitHub Copilot inline
  integration, Antigravity entrypoints (`inline`, `aux_pane`, `global`, `file`),
  host theming, and CSP boundaries.
- `testing-and-packaging.md` — unit/protocol harnesses, MCPJam Inspector /
  `basic-host` / Playwright setup, `.mcp.json` (Copilot/portable), and
  Antigravity plugin bundling (`plugin.json`, `mcp_config.json`,
  `agy plugin validate`).
