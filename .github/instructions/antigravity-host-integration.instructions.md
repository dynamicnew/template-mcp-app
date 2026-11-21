---
applyTo: "src/**,server/**,ui/**,plugin.json,.antigravity-plugin/**,mcp_config.json"
description: Google Antigravity MCP App extended layer: com.google.antigravity/ui entrypoints (inline, aux_pane, global, file), file viewer APIs, host limits, plugin packaging, and CLI caveats.
---

- **Antigravity UI entrypoints (`_meta["com.google.antigravity/ui"].entrypoints`):**
  - **Inline in chat:** Omit `aux_pane`/`global`/`file` entrypoints. Inline
    cards auto-resize up to 800 px (falling back to 150 px if no `size-changed`
    arrives within 100 ms; design taller views to scroll internally).
  - **`aux_pane` (side panel):** Opens beside the active conversation (one tab
    per `(server, tool)`). Manual launches pass `{}`; repeat agent calls update
    the open tab in place. Does not render inline in Antigravity (other hosts
    ignore `com.google.antigravity/ui` and still render `_meta.ui.resourceUri`
    inline).
  - **`global` (full page):** Opens full-width outside any conversation from
    Plugins in the left sidebar with `{}`. Conversation-only capabilities
    (`message`, `updateModelContext`) are omitted; guard calls to them.
  - **`file` (custom file viewer/editor):** Always place `file` entrypoints on
    their own separate tool — Antigravity unconditionally hides `file` tools
    from the model and passes `{"file": {"name", "resourceUri"}}`. `extensions`
    is required (`string[]`, normalized lowercase with leading dot). Read via
    `resources/read` on that opaque `antigravity-file://` URI (base64
    `application/octet-stream`, max 10 MiB); enable writes only when
    `hostContext["com.google.antigravity/resources"]?.write` is truthy via
    `com.google.antigravity/resources/write` (`{ uri, text }` or `{ uri, blob }`,
    max 10 MiB).
- **Host limits & graceful degradation:**
  - Keep `ui://` HTML documents and `antigravity-file://` reads/writes <= 10 MiB
    and persisted `CallToolResult` payloads <= 1 MiB (load larger datasets via
    an app-only `visibility: ["app"]` helper tool).
  - Stay within 4 concurrent `tools/call` requests per view (scoped to the
    owning server; aborted on close) and account for up to 5 hidden
    `aux_pane`/`global` frames staying mounted (LRU).
  - Iframes have **no persistent browser storage** and Antigravity sends **no**
    `ui/resource-teardown` notification on close: persist edits eagerly.
  - Handle JSON-RPC `-32601` gracefully for Antigravity-only host actions
    (`com.google.antigravity/toggle-aux-pane`,
    `_meta["com.google.antigravity/message"]`) so the same bundle runs cleanly in
    Copilot and other hosts.
- **Antigravity plugin packaging (`plugin.json` + `mcp_config.json`):**
  - Place `plugin.json` at `<plugin-root>/plugin.json` (or
    `.antigravity-plugin/plugin.json` in multi-tool repos) with
    `"$schema": "https://antigravity.google/schemas/v1/plugin.json"`, lowercase
    kebab-case `name` (namespaces servers as `<plugin-name>_<server-key>`), and
    brand-only `displayName`.
  - Installed plugins unpack into a read-only folder replaced on updates. In
    `mcp_config.json`, use `${PLUGIN_ROOT}` for read-only bundled files and
    `${PLUGIN_DATA}` for writable state (never hardcode user paths or redeclare
    `PLUGIN_ROOT` / `PLUGIN_DATA` inside `"env"`). Keep contributor scaffolding
    (root `AGENTS.md`, `.agents/`, `.github/`, `scripts/check-context-size.ps1`)
    separate from shipped plugin files.
- **Antigravity CLI caveat:** Antigravity CLI ignores `_meta.ui.visibility` and
  exposes all tools as plain text — validate inputs and authorize server-side.
