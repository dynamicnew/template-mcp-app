---
name: generate-handoff-canvas
description: Generate or recreate a project-local handoff canvas panel (GitHub Copilot SDK extension under .github/extensions/<canvas-id>/ or Antigravity MCP App side-pane plugin under .agents/plugins/<canvas-id>/) without shipping prebuilt panel code in the template.
---

# Generate a project handoff canvas panel

Build a self-contained, source-backed handoff panel that reads Git state and
`docs/ai/handoff.md` (and optionally edits `docs/ai/todo.md`). The panel is a
presentation of repository docs — never a second handoff record.

Ask which host target is needed if unclear: **Copilot canvas extension**
(`.github/extensions/<canvas-id>/`), **Antigravity MCP App side pane**
(`.agents/plugins/<canvas-id>/`), or both.

## Target A: Copilot project-local canvas (`.github/extensions/<canvas-id>/`)

Place all files directly in a child folder under `.github/extensions/`
(`.github/extensions/<canvas-id>/extension.mjs` — 3 parent hops back to the Git
root; `<canvas-id>` is lowercase kebab-case, e.g. `<project>-handoff`). No
`package.json`, `npm install`, VSIX, or manifest is required: the Copilot host
discovers `extension.mjs` automatically and supplies
`@github/copilot-sdk/extension`. Git must be on `PATH`.

### Files to generate

| File | Responsibility |
|---|---|
| `settings.mjs` | Exports `CANVAS_ID`, `PROJECT_NAME`, `BASE_BRANCH` (default `"main"`), `HANDOFF_PATH` (`"docs/ai/handoff.md"`), `TODO_PATH` (`"docs/ai/todo.md"`), and `MAX_HANDOFF_BYTES` (`256 * 1024`). |
| `extension.mjs` | Resolves `ROOT = path.resolve(path.dirname(fileURLToPath(import.meta.url)), "..", "..", "..")`; runs one ephemeral loopback HTTP server per `instanceId` (`server.listen(0, "127.0.0.1")`); registers `createCanvas({ id, displayName, description, actions: [get_status], open, onClose })` and calls `await joinSession({ canvases: [canvas], commands })` from `@github/copilot-sdk/extension`. Closes server + `closeAllConnections()` in `onClose`. |
| `status.mjs` | Concurrent read-only `execFile("git", ..., { cwd: root, timeout: 10_000, windowsHide: true, maxBuffer: 1024 * 1024 })` (no shell): current branch, `status --short`, `status --porcelain=v1 -z` (skipping the extra rename/copy path after `[RC]`), recent commits, upstream, `remote get-url origin` sanitized to `https://github.com/<owner>/<repo>` (discard all credentials/non-GitHub URLs), ahead/behind vs `BASE_BRANCH`, and handoff freshness (`git log -1` + `rev-list --count <last>..HEAD`). Reads `HANDOFF_PATH` up to `MAX_HANDOFF_BYTES`; returns diagnostics instead of throwing. |
| `links.mjs` | Validated OS opener for `POST /api/open` (`{ url }`). Permits only `http:`, `https:`, `mailto:`, `vscode://file/...`, and `file://` for a strict non-executable binary allowlist (`png|jpg|jpeg|gif|webp|ico|pdf|zip|mp3|mp4|wav`). Verifies `realpath(root)` + `realpath(target)` containment, rejects control chars, double-encoding, credentials, and Windows reserved device names (`con`, `prn`, `aux`, `nul`, `com1-9`, `lpt1-9`). **Strip `ELECTRON_*` env vars** (`ELECTRON_RUN_AS_NODE`) before spawning `shell: false` (Windows: PowerShell `-EncodedCommand` `Start-Process`; macOS: `/usr/bin/open`; Linux: `xdg-open`, 10 s timeout). |
| `ui.mjs` | Exports `PAGE = String.raw\`<!doctype html>...\`` with zero remote fonts/scripts and `__HANDOFF_ACTION_TOKEN__` placeholder replaced once per server instance. Builds all DOM strictly with `document.createElement` + `textContent` (never `innerHTML` on repo/Git text). Renders sticky header + Refresh, 4 snapshot cards (Branch, Working tree, Origin, Handoff file), freshness banner (green `0`, amber `1-4`, alert `5+` commits since handoff change), collapsible commits/changes lists, sticky TOC + minimal Markdown renderer (skipping fenced code headings), quick-open buttons, dark/light (`prefers-color-scheme`), `prefers-reduced-motion`, and no horizontal overflow at `380px`. |
| `README.md` | Usage, safety invariants, file table, and verification commands. |
| `verify/panel-smoke.mjs` | Offline smoke test using a temporary stub `@github/copilot-sdk/extension`, canned `git` shim, security checks (404 wrong capability, 403 bad host, 403 missing token), and Playwright dark/light/`380px` checks. |

### Optional modules (add only when requested)

- `canvas-launcher.mjs`: Registers a `/canvases` slash command using
  `session.rpc.canvas.list()`, `session.capabilities.ui?.elicitation`,
  `session.ui.select(...)`, and `session.rpc.canvas.open(...)` with a
  deterministic `instanceId` (`base64url` of `extensionId\0canvasId`) so
  reopening focuses the existing panel.
- `notes.mjs`: Fixed-path editor for `TODO_PATH` (`docs/ai/todo.md`, max 64 KiB)
  at `GET`/`PUT /api/notes`. Rejects symlinks and hardlinks (`lstat`,
  `nlink === 1n`, inode check), uses SHA-256 optimistic concurrency (HTTP 409
  on stale save), serializes in-process saves, uses an exclusive lockfile
  (never auto-stolen), and writes via atomic temp-file rename. Never executes
  TODO text or touches the handoff.
- `actions.mjs` / `clarifydoc.mjs`: Explicitly allowlisted, checkbox-confirmed
  project commands or read-only ClarifyDoc MCP queries; never expose arbitrary
  commands, raw secrets, or private launch URLs.

### Non-negotiable security invariants

1. Generate two 32-byte hex secrets (`randomBytes(32).toString("hex")`) per
   instance: a URL path `capability` (`/<capability>/`) and a same-origin
   `token` (`x-handoff-token`). Validate both with `timingSafeEqual`.
2. Check `req.headers.host` equals `127.0.0.1:<port>` or `localhost:<port>`
   before serving any route (`403` otherwise); return `404` on capability
   mismatch.
3. Require `x-handoff-token`, `origin === "http://" + host`, and
   `content-type === "application/json"` on every `POST`/`PUT` route (`403`
   otherwise), plus bounded request body size (`<= 8192` bytes for opens/actions).
4. Set `cache-control: no-store`, `x-content-type-options: nosniff`,
   `referrer-policy: no-referrer`, and
   `content-security-policy: frame-ancestors 'self' vscode-webview: vscode-file: https://*.vscode-cdn.net`
   on the HTML page.

## Target B: Antigravity MCP App side-pane panel (`.agents/plugins/<canvas-id>/`)

Antigravity does not load `.github/extensions/*/extension.mjs`. To build an
Antigravity side-pane or full-page handoff panel, follow Antigravity's built-in
`mcp-app-development` skill:

1. Create a workspace plugin folder `.agents/plugins/<canvas-id>/` with
   `plugin.json`, `mcp_config.json` (Stdio server using `${PLUGIN_ROOT}` in
   `args` and `${PLUGIN_DATA}` for any writable state — never relative CWD,
   literal secrets, or hardcoded user paths), and a small server + single-file
   HTML UI.
2. Reuse the same read-only `status.mjs` Git/handoff reader (and optional
   locked `notes.mjs` TODO reader/writer), placing them directly inside
   `.agents/plugins/<canvas-id>/` when generating Target B standalone without
   Target A, and keeping every file-path and credential-scrubbing invariant
   above.
3. Expose an MCP resource `ui://<canvas-id>/handoff.html` with MIME type
   `text/html;profile=mcp-app`, built as one self-contained HTML file with
   `@modelcontextprotocol/ext-apps` (zero remote scripts/fonts, DOM `textContent`
   rendering).
4. Point an entry tool (accepting empty `{}` input) at that resource with
   `_meta.ui.resourceUri` and
   `_meta["com.google.antigravity/ui"].entrypoints` (`aux_pane` and optionally
   `global`), plus app-only (`visibility: ["app"]`) refresh/save tools if
   needed.

## Verification before completion

1. Run the panel smoke test against a stub host and `git` shim (inside Docker
   when the project requires Docker-only tests).
2. Confirm capability rejection (`404`), bad `Host` rejection (`403`), missing
   action-token rejection (`403`), credential scrubbing, dark/light/narrow
   (`380px`) rendering with zero console/page errors, and clean server shutdown.
3. Report unverified live-host behavior (such as native OS link opening or host
   canvas registration) plainly.
