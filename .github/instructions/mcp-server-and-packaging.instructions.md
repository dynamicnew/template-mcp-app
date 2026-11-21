---
applyTo: "server/**,src/**,main.ts,server.ts,plugin.json,.antigravity-plugin/**,mcp_config.json,package.json"
description: Rules for MCP App server transports (stdio/Streamable HTTP), tool design, Antigravity plugin packaging (plugin.json, mcp_config.json), path variables, and auth.
---

- **Server transport & runtime:**
  - Local stdio servers: default to `tsx` in development and `"command": "node"`
    in shipped `mcp_config.json` so Antigravity can fall back to its bundled
    Node.js runtime across macOS, Linux, and Windows. Never write logs to
    `stdout` on stdio servers (use `stderr`); `stdout` carries JSON-RPC frames
    exclusively.
  - Remote servers: use Streamable HTTP via `serverUrl` (or `url`); legacy
    HTTP+SSE and `httpUrl` are unsupported.
- **Antigravity plugin packaging (`plugin.json` + `mcp_config.json`):**
  - Place `plugin.json` at `<plugin-root>/plugin.json` (or
    `.antigravity-plugin/plugin.json` in multi-tool repos). Include
    `"$schema": "https://antigravity.google/schemas/v1/plugin.json"`, lowercase
    kebab-case `name` (`^[a-z0-9]+(-[a-z0-9]+)*$`), and brand-only `displayName`
    (no `Antigravity`/`Gemini`/`Claude`/`Cursor` prefix or `Plugin`/`MCP` suffix).
  - Antigravity namespaces plugin servers as `<plugin-name>_<server-key>`.
  - Installed plugins unpack into a read-only directory replaced on updates. Use
    `${PLUGIN_ROOT}` for read-only bundled files and `${PLUGIN_DATA}` for
    writable state outside the workspace in `mcp_config.json` (`args`, `cwd`,
    custom `env`). Never hardcode user paths or redeclare `PLUGIN_ROOT` /
    `PLUGIN_DATA` as keys inside `"env"` (Antigravity exports both automatically).
  - Keep contributor scaffolding (root `AGENTS.md`, `.agents/`, `.github/`,
    `scripts/check-context-size.ps1`) separate from shipped plugin files.
- **Auth & credentials:** Never commit API keys, tokens, or OAuth client
  secrets. Prefer OAuth 2.0 + PKCE (CIMD/DCR or manual `oauth.clientId`),
  `"authProviderType": "google_credentials"` for Google ADC, or per-server
  `mcpServers.<server>.variables` (never at the root of `mcp_config.json`; note
  that declaring `variables` leaves CLI installs disabled until configured).
- **Tool design:** Validate all inputs with strict Zod schemas, set accurate
  tool annotations (`readOnlyHint`, `destructiveHint`, `idempotentHint`), return
  actionable errors (`isError: true`), and return concise plain-text `content`
  alongside any `structuredContent`.
