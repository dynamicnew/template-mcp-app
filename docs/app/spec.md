# MCP App specification (fill in)

Read this before adding or modifying MCP tools, `ui://` HTML resources,
GitHub Copilot or Google Antigravity host integrations, CSP/CORS policies, or
packaging. Keep internal open questions in `docs/ai/`, not in user-facing UI
copy, READMEs, or tool descriptions.

## Identity and packaging

- **App / server `name` (kebab-case):** `<!-- e.g., release-board -->`
- **`displayName`:** `<!-- e.g., Release Board (no Antigravity/Gemini/Claude/Cursor prefix; no Plugin/MCP suffix) -->`
- **Current `version`:** `<!-- e.g., 0.1.0 -->`
- **UI framework & bundler:** `<!-- React / Vanilla TS / Vue / Svelte / Preact / Solid + Vite + vite-plugin-singlefile -->`
- **Server transport:** `<!-- local stdio (`node`) | remote Streamable HTTP (`/mcp`) | dual stdio + HTTP -->`
- **Host distribution targets:**
  - **GitHub Copilot (VS Code & CLI):** `<!-- .mcp.json ("mcpServers") | Copilot Global (~/.copilot/mcp-config.json) | M365 Copilot HTTP -->`
  - **Google Antigravity (App & CLI):** `<!-- plugin.json / .antigravity-plugin/plugin.json + mcp_config.json (${PLUGIN_ROOT}/${PLUGIN_DATA}) -->`
  - **Other MCP Apps hosts (optional):** `<!-- Claude Desktop / Goose / Postman / hybrid standalone web app -->`

## Tools, `ui://` resources, and host matrix

| Tool name | Role & visibility | `ui://` resource URI | Copilot behavior (`title` / `readOnlyHint` / disambiguation) | Antigravity entrypoint (`inline` / `aux_pane` / `global` / `file`) |
|---|---|---|---|---|
| `<!-- list_items -->` | Data (`model` + `app`) | *(none — no re-render)* | `readOnlyHint: true`; description distinguishes from built-in `#todos` | Standard tool (plain text + `structuredContent`) |
| `<!-- show_board -->` | Render (`model` + `app`) | `ui://<app>/board.html` | Inline in chat (~768×400 px); `readOnlyHint: true` | `inline` or `aux_pane` / `global` (accepts `{}` for manual launch) |
| `<!-- update_item -->` | Mutate (`model` + `app` or `["app"]`) | *(none)* | Confirmation dialog with editable args; strict Zod + `isError` | Standard or `["app"]` helper (still validated server-side for CLI) |
| `<!-- view_custom_file -->` | File viewer *(host-only)* | `ui://<app>/viewer.html` | Ignored / hidden unless called as standard tool | Separate `file` tool (`extensions: [".ext"]`); max 10 MiB read/write |

## CSP, CORS, auth, and state boundaries

- **CSP `_meta.ui.csp` (set inside `contents[]`):**
  - `connectDomains`: `<!-- [] preferred (route calls through tools/call); bare origins only if required -->`
  - `resourceDomains`: `<!-- [] preferred (bundle assets via vite-plugin-singlefile) -->`
  - `frameDomains`: `<!-- [] unless embedding vetted nested iframes -->`
- **CORS `_meta.ui.domain` (optional):** `<!-- Omit unless calling an origin-allowlisted API directly from the iframe -->`
- **Authentication mode:** `<!-- None | OAuth 2.0 PKCE | Google ADC (Antigravity) | mcpServers.<server>.variables / VS Code inputs -->`
- **Persistent state (`${PLUGIN_DATA}` or server storage):** `<!-- Re-renders reset UI state in Copilot; Antigravity has no persistent browser storage and sends no teardown event — persist eagerly -->`
- **Security boundaries:** `<!-- Zod validation on every tool, server-side authorization on visibility:["app"] and file tools, textContent rendering, zero secrets in templates -->`

## Cross-host release checklist

- [ ] Dependencies use `@modelcontextprotocol/ext-apps` v2 + split `@modelcontextprotocol/*` v2 SDKs (no legacy `@modelcontextprotocol/sdk` v1); server `tsconfig` sets `moduleResolution: "bundler"` and `"DOM"` in `lib`.
- [ ] All `App` handlers (`ontoolinput`, `ontoolinputpartial`, `ontoolresult`, `ontoolcancelled`, `onhostcontextchanged`, `onteardown`) register before `app.connect()`.
- [ ] Single-file `ui://` HTML bundle stays `<= 10 MiB`, serves `text/html;profile=mcp-app`, sets `_meta.ui.csp` inside `contents[]`, uses CSS variable fallbacks, and supports Light/Dark + `prefers-reduced-motion`.
- [ ] Every tool returns concise plain-text `content` for non-UI CLI hosts alongside `structuredContent`.
- [ ] Copilot layer verified: server `instructions` and tool descriptions disambiguate from built-in tools; `readOnlyHint` and confirmation dialogs behave as intended; inline layout fits `<= 768 px` wide without horizontal overflow.
- [ ] Antigravity layer verified (when packaged as a plugin): `file` entrypoints live on separate tools; optional host actions handle `-32601`; `plugin.json` + `mcp_config.json` (`${PLUGIN_ROOT}` / `${PLUGIN_DATA}`) pass `agy plugin validate`.
- [ ] `mcp-release-qa` and `mcp-implementation-security-review` completed with recorded evidence.
