---
applyTo: "ui/**,web/**,src/**,*.html,vite.config.*"
description: Rules for building interactive MCP Apps with @modelcontextprotocol/ext-apps v2, Vite single-file bundling, lifecycle handlers, CSP/CORS, styling, and non-UI fallbacks.
---

- **SDK & dependency hygiene:** Use `@modelcontextprotocol/ext-apps` v2 with the
  split base MCP SDK v2 packages (`@modelcontextprotocol/client`, `server`,
  `node`, `express` at `^2.0.0`, plus `zod@^4.2.0`). Never add the legacy
  `@modelcontextprotocol/sdk` v1 package. Always install packages via your
  package manager (`npm install`) instead of hand-writing versions from memory.
- **Tool + resource pairing:** Register every UI-backed tool with
  `registerAppTool` pointing `_meta.ui.resourceUri` to a `ui://` resource
  registered with `registerAppResource` returning MIME type
  `text/html;profile=mcp-app` (`RESOURCE_MIME_TYPE`). Bundle each view into a
  single self-contained HTML document via `vite-plugin-singlefile`.
- **Lifecycle order:** Register ALL `App` event handlers (`ontoolinput`,
  `ontoolinputpartial`, `ontoolresult`, `onhostcontextchanged`, `onteardown`)
  BEFORE calling `app.connect()`. Handle `ontoolinputpartial` for progressive
  rendering of large/streaming inputs, and use `app.sendLog()` for host-visible
  diagnostics rather than relying solely on iframe console logs.
- **CSP & CORS placement:** MCP App HTML runs in a sandboxed iframe with no
  same-origin server and default-deny CSP (even `localhost` requires an
  allowlist). Always place `_meta.ui.csp` (`connectDomains`, `resourceDomains`)
  and optional `_meta.ui.domain` inside the `contents[]` objects returned by
  `registerAppResource()`'s read callback — never in `registerAppResource()`'s
  config object. Prefer routing backend calls through `tools/call` over
  broadening `connectDomains`; allowlist bare origins only when required.
- **Theming, security & non-UI fallback:** Apply host styling with
  `applyDocumentTheme`, `applyHostStyleVariables`, and `applyHostFonts` (or
  React `useHostStyles*` hooks); support Light/Dark themes and
  `prefers-reduced-motion`. Render untrusted or repo text with `textContent`
  (never raw `innerHTML`). Always return clear plain-text `content` alongside
  `structuredContent` so non-UI clients work cleanly.
