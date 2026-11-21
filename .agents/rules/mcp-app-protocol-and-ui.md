---
trigger: model_decision
description: Cross-host rules for building interactive MCP Apps with @modelcontextprotocol/ext-apps v2, Vite single-file bundling, lifecycle handlers, CSP/CORS, theming, and non-UI fallbacks.
---

- **SDK & TypeScript setup:** Use `@modelcontextprotocol/ext-apps` v2 with the
  split base MCP SDK v2 packages (`@modelcontextprotocol/client`, `server`,
  `node`, `express` at `^2.0.0`, plus `zod@^4.2.0`). Never add the legacy
  `@modelcontextprotocol/sdk` v1 package. Install dependencies via your package
  manager (`npm install`) instead of hand-writing versions. In server
  `tsconfig.json` files importing `@modelcontextprotocol/ext-apps`, include
  `"moduleResolution": "bundler"` and `"DOM"` in `lib`.
- **Tool + resource architecture:** Separate data tools from UI render tools so
  data calls do not needlessly re-render views. Link each UI tool via
  `registerAppTool` (`_meta.ui.resourceUri`) to a `ui://` resource registered
  with `registerAppResource` returning `RESOURCE_MIME_TYPE`
  (`text/html;profile=mcp-app`). A mismatched or missing resource silently falls
  back to plain text. Bundle each view as one self-contained HTML file with
  `vite-plugin-singlefile`.
- **Payload & context hygiene:** Keep `content` to a concise plain-text summary
  for the model and non-UI hosts; pass UI display payloads in
  `structuredContent`. Paginate large datasets and load details on demand via
  `visibility: ["app"]` helper tools called with `app.callServerTool()`.
- **Lifecycle order:** Register ALL `App` handlers (`ontoolinput`,
  `ontoolinputpartial`, `ontoolresult`, `ontoolcancelled`,
  `onhostcontextchanged`, `onteardown`) BEFORE calling `app.connect()`. Use
  `ontoolinputpartial` for progressive rendering of streamed inputs and
  `app.sendLog()` for host-visible diagnostics.
- **CSP & CORS placement:** MCP App views run in sandboxed iframes under a
  default-deny CSP with no same-origin server (even `localhost` needs an
  allowlist). Always place `_meta.ui.csp` (`connectDomains`, `resourceDomains`,
  `frameDomains`) and optional `_meta.ui.domain` inside the `contents[]` objects
  returned by `registerAppResource()`'s read callback — never in
  `registerAppResource()`'s config object. Prefer routing backend calls through
  `tools/call` over broadening `connectDomains`; declare bare origins only.
- **Theming, a11y & security:** Apply host styles with `applyDocumentTheme`,
  `applyHostStyleVariables`, and `applyHostFonts` (or React `useHostStyles*`
  hooks) with safe CSS fallbacks (for example,
  `var(--color-text-primary, CanvasText)`). Support Light/Dark themes and
  `prefers-reduced-motion`. Render untrusted text with `textContent` (never raw
  `innerHTML`), validate all tool inputs and authorize `visibility: ["app"]`
  tools server-side, and never store secrets in HTML templates.
