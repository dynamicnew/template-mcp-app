---
trigger: model_decision
description: Guardrails for creating, editing, or reviewing a project-local handoff canvas panel (.github/extensions/<canvas-id>/ or .agents/plugins/<canvas-id>/).
---

- Load the `generate-handoff-canvas` skill before creating a new handoff canvas
  panel from scratch.
- Keep `docs/ai/handoff.md` (and `docs/ai/todo.md`) as the sole source of
  truth; the panel renders them and never creates a parallel handoff record.
- For Copilot canvas extensions (`.github/extensions/<canvas-id>/extension.mjs`),
  keep the exact folder depth (3 parent hops back to Git root; no `package.json`
  or VSIX), bind only `127.0.0.1:0` per `instanceId`, and close all connections
  in `onClose`. Enforce `Host` validation (`403`), `timingSafeEqual` capability
  URL (`404`), and same-origin `x-handoff-token` + `Origin` + JSON
  `Content-Type` on all mutating routes (`403`).
- Build Copilot panel UI exclusively via DOM APIs and `textContent` (no
  `innerHTML` on repository/Git text; zero remote fonts or scripts), scrub
  credentials from Git remote URLs, validate OS link targets with `realpath`
  containment + a non-executable allowlist, use `spawn` with `shell: false`,
  and strip `ELECTRON_*` env vars before launching OS link handlers.
- For Antigravity side-pane panels (`.agents/plugins/<canvas-id>/`), use an MCP
  App (`ui://` `text/html;profile=mcp-app` + `@modelcontextprotocol/ext-apps` +
  `aux_pane` entrypoint) referencing `${PLUGIN_ROOT}`, with zero literal secrets
  or machine-specific paths.
- Verify security guards and dark/light/`380px` layout with an offline smoke
  test before claiming the panel works; report live-host limits plainly.
