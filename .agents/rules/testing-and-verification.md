---
trigger: model_decision
description: Cross-host testing pyramid for MCP Apps: unit/protocol tests, single-file UI/CSP checks, MCPJam Inspector & basic-host emulation, VS Code Copilot Chat checks, and agy plugin validate.
---

- **Layer 1 — Deterministic unit & protocol tests:** Run offline unit, Zod
  validation, and real MCP protocol session tests locally or in Docker using
  disposable fixtures (never mutating real user accounts or `~/.gemini/` /
  `~/.copilot/` state). Cover valid calls, invalid schemas, missing IDs, empty
  datasets, large-result pagination, and plain-text `content` fallbacks.
- **Layer 2 — Single-file `ui://` bundle & CSP checks:** Verify the Vite
  `vite-plugin-singlefile` HTML build stays `<= 10 MiB`, serves MIME type
  `text/html;profile=mcp-app`, places `_meta.ui.csp` inside `contents[]`, and
  passes Light/Dark theme, narrow-viewport wrapping, `prefers-reduced-motion`,
  and zero console/page-error checks.
- **Layer 3 — Emulated host inspection (`MCPJam` / `basic-host`):**
  - Use **MCPJam Inspector** (`npx @mcpjam/inspector@latest` or CLI replay)
    and/or the official **`basic-host`** reference harness
    (`SERVERS='["http://localhost:3001/mcp"]'`) to inspect `ui/initialize`,
    `tool-input`, `tool-result`, and `tools/call` traces, test Strict CSP mode,
    and run fixed-input tool calls deterministically before chat testing.
- **Layer 4 — Real-host qualification (Copilot + Antigravity):**
  - **VS Code GitHub Copilot Chat:** Verify tool selection alongside built-in
    tools (like `#todos`), `readOnlyHint: true` auto-run vs. editable-argument
    confirmation dialogs, `MCP: List Servers` → `Show Output`, and DevTools
    console for CSP errors.
  - **Google Antigravity:** Validate plugin bundles with
    `agy plugin validate --strict <path>` (clean bundle) or
    `agy plugin validate --json <path> | jq -e ...` (repo root with contributor
    `AGENTS.md`/other-tool files), qualify `agy plugin install <path>`, and
    verify declared entrypoints (`inline`, `aux_pane`, `global`, `file`) in the
    Antigravity App. Close/reopen views after HTML edits; restart local stdio
    servers in Customizations after server code edits.
- **Honest evidence tiers:** Report unit/protocol passes, `MCPJam`/`basic-host`
  emulation, `agy plugin validate`, VS Code Copilot checks, and live Antigravity
  App checks separately. Never claim one host's pass proves another host works.
  Use `mcp-release-qa` before shipping a release.
