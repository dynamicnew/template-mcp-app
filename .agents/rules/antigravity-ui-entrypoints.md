---
trigger: model_decision
description: Rules for Google Antigravity MCP App UI entrypoints (inline, aux_pane, global, file), custom file viewer/editor APIs, host limits, and CLI fallback behavior.
---

- **Antigravity UI entrypoints (`_meta["com.google.antigravity/ui"].entrypoints`):**
  - **Inline in chat:** Omit `aux_pane`/`global`/`file` entrypoints. Inline
    cards auto-resize up to 800 px (falling back to 150 px if no `size-changed`
    arrives within 100 ms; design taller views to scroll internally).
  - **`aux_pane` (side panel):** Opens beside the active conversation (one tab
    per `(server, tool)`). Manual launches pass `{}`; repeat agent calls update
    the open tab in place. Does not render inline.
  - **`global` (full page):** Opens full-width outside any conversation from
    Plugins in the left sidebar with `{}`. Conversation-only capabilities
    (`message`, `updateModelContext`) are omitted; do not assume they exist.
  - **`file` (custom file viewer/editor):** Always place `file` entrypoints on
    their own separate tool — Antigravity unconditionally hides any `file` tool
    from the model regardless of `visibility` and passes
    `{"file": {"name", "resourceUri"}}`. `extensions` is required (`string[]`,
    normalized lowercase with leading dot). Read via `resources/read` for that
    exact opaque `antigravity-file://` URI (base64 `application/octet-stream`,
    max 10 MiB); enable writes only when
    `hostContext["com.google.antigravity/resources"]?.write` is truthy via
    `com.google.antigravity/resources/write` (`{ uri, text }` or `{ uri, blob }`,
    max 10 MiB).
- **Antigravity host limits & persistence:**
  - Keep `ui://` HTML documents and `antigravity-file://` reads/writes <= 10 MiB
    and persisted `CallToolResult` payloads <= 1 MiB (load larger datasets via
    an app-only `visibility: ["app"]` helper tool).
  - Stay within 4 concurrent `tools/call` requests per view (scoped to the
    owning server; in-flight calls abort on close) and account for up to 5
    hidden `aux_pane`/`global` frames staying mounted (LRU).
  - Iframes have **no persistent browser storage** and Antigravity sends **no**
    `ui/resource-teardown` notification on close: persist user edits eagerly
    through server tools or `com.google.antigravity/resources/write`.
  - Handle JSON-RPC `-32601` gracefully when calling optional Antigravity host
    actions (`com.google.antigravity/toggle-aux-pane`) or passing
    `_meta["com.google.antigravity/message"]` (`{ target: "new" }`).
- **Non-UI surface caveat:** Antigravity CLI and IDE run tools as plain-text MCP
  without mounting iframes, and the CLI ignores `_meta.ui.visibility` (exposing
  `visibility: ["app"]` and `file` tools to the model). Always validate inputs
  and enforce authorization server-side.
