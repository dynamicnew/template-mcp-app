# MCP App specification (fill in)

Read this before adding or modifying MCP tools, `ui://` HTML resources,
Antigravity UI entrypoints, CSP/CORS policies, or plugin packaging. Keep
internal open questions in `docs/ai/`, not in user-facing UI copy or tool
descriptions.

## Identity and packaging

- **App / plugin `name` (kebab-case):** `<!-- e.g., release-board -->`
- **`displayName`:** `<!-- e.g., Release Board (no Antigravity/Gemini/Claude/Cursor prefix; no Plugin/MCP suffix) -->`
- **Current `version`:** `<!-- e.g., 0.1.0 -->`
- **UI framework & bundler:** `<!-- React / Vanilla TS / Vue / Svelte / Preact / Solid + Vite + vite-plugin-singlefile -->`
- **Server transport:** `<!-- local stdio (`node`) | remote Streamable HTTP (`serverUrl`) | dual stdio + HTTP -->`
- **Packaging target:** `<!-- Antigravity plugin (plugin.json / .antigravity-plugin/plugin.json + mcp_config.json) | standalone MCP server | hybrid web + MCP App -->`
- **Target hosts:** `<!-- Antigravity App (inline / aux_pane / global / file), Antigravity CLI (plain-text fallback), VS Code, Claude Desktop -->`

## Tools and `ui://` resource inventory

| Tool name | Visibility | Antigravity entrypoint | `ui://` resource URI | Input / payload notes |
|---|---|---|---|---|
| `<!-- show_board -->` | `model` + `app` | `inline` / `aux_pane` / `global` | `ui://<app>/<view>.html` | `<!-- Accepts {} for manual aux_pane/global launch; <= 1 MiB CallToolResult -->` |
| `<!-- fetch_page -->` | `["app"]` | *(none — helper)* | *(none)* | `<!-- App-only helper for datasets > 1 MiB or polling; still validated server-side for CLI -->` |
| `<!-- view_custom_file -->` | *(host-only)* | `file` (`[".ext"]`) | `ui://<app>/viewer.html` | `<!-- Separate tool; receives {"file":{"name","resourceUri"}}; max 10 MiB read/write -->` |

## CSP, CORS, auth, and state boundaries

- **CSP `_meta.ui.csp` (set inside `contents[]`):**
  - `connectDomains`: `<!-- [] preferred (route API calls through tools/call); bare origins only if required -->`
  - `resourceDomains`: `<!-- [] preferred (bundle all assets via vite-plugin-singlefile) -->`
- **CORS `_meta.ui.domain` (optional):** `<!-- Omit unless calling an origin-allowlisted API directly from the iframe -->`
- **Authentication mode:** `<!-- None | OAuth 2.0 PKCE | Google ADC | mcpServers.<server>.variables -->`
- **Persistent state (`${PLUGIN_DATA}` or backend):** `<!-- Remember Antigravity iframes have no persistent browser storage and send no ui/resource-teardown; persist edits eagerly -->`
- **Security boundaries:** `<!-- Read-only vs mutating tools, Zod validation, server-side auth checks (CLI ignores _meta.ui.visibility), untrusted text rendered via textContent -->`

## Quality and release checklist

- [ ] Dependencies use `@modelcontextprotocol/ext-apps` v2 + split `@modelcontextprotocol/*` v2 SDKs (no legacy `@modelcontextprotocol/sdk` v1).
- [ ] All `App` lifecycle handlers (`ontoolinput`, `ontoolinputpartial`, `ontoolresult`, `onhostcontextchanged`, `onteardown`) are registered before `app.connect()`.
- [ ] Single-file `ui://` HTML bundle builds cleanly, stays `<= 10 MiB`, uses MIME type `text/html;profile=mcp-app`, and respects host theme tokens, Light/Dark modes, and `prefers-reduced-motion`.
- [ ] Every UI-linked tool returns clear plain-text `content` alongside `structuredContent` for non-UI clients.
- [ ] File viewer tools (`type: "file"`) are separate from model-callable tools and gate writes on `hostContext["com.google.antigravity/resources"]?.write`.
- [ ] When packaged as an Antigravity plugin: `plugin.json` includes `$schema`, `mcp_config.json` uses `${PLUGIN_ROOT}` / `${PLUGIN_DATA}` (never hardcoded paths or duplicated in `env`), and `agy plugin validate` passes.
