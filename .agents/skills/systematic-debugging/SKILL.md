---
name: systematic-debugging
description: Investigate cross-host MCP App failures—lifecycle handler races, single-file ui:// bundling, CSP/CORS blocks, VS Code GitHub Copilot tool routing or approval issues, Antigravity entrypoints (inline/aux_pane/global/file), stdio/HTTP transport errors, and plugin packaging—before changing code.
---

# Systematic debugging

Find the root cause of the observed MCP App failure, not just a workaround.
Follow [AGENTS.md](../../../AGENTS.md) and never loosen CSP/schemas, swallow
errors, or commit hardcoded credentials/paths.

## Establish the actual host surface and build

- Read `#current-handoff` in [docs/ai/handoff.md](../../../docs/ai/handoff.md)
  and relevant sections of [docs/app/spec.md](../../../docs/app/spec.md).
- Confirm which host surface failed: unit/protocol test, Vite single-file build,
  MCPJam Inspector / `basic-host`, VS Code GitHub Copilot Chat, Copilot CLI,
  `agy plugin validate`, Antigravity App (`inline`/`aux_pane`/`global`/`file`),
  or Antigravity CLI (`agy`).
- Confirm you are testing the fresh build, not a stale copy: rebuild the
  single-file HTML bundle, close and reopen `ui://` views for HTML edits,
  restart local stdio servers (Customizations in Antigravity or `MCP: List
  Servers` in VS Code) after server edits, and reinstall after manifest edits.

## Collect evidence by layer

1. **SDK & lifecycle (`@modelcontextprotocol/ext-apps`):**
   - Trace the handshake in MCPJam Inspector (`ui/initialize` →
     `ui/notifications/tool-input` → `ui/notifications/tool-result` →
     `tools/call`). Missing `ui/initialize` means `useApp` / `app.connect()`
     never ran; missing events mean handlers were registered AFTER
     `app.connect()`.
   - Verify split MCP SDK v2 packages (`@modelcontextprotocol/client`, `server`,
     `node`, `express`) with zero v1 SDK conflicts, and confirm server
     `tsconfig.json` includes `moduleResolution: "bundler"` and `"DOM"` in `lib`.
   - Use `app.sendLog()` to surface iframe errors to the host.
2. **`ui://` bundle & CSP/CORS:**
   - Verify `registerAppResource` URI matches `_meta.ui.resourceUri` exactly
     (mismatches silently fall back to text), returns `text/html;profile=mcp-app`,
     stays `<= 10 MiB`, and sets `_meta.ui.csp` / `_meta.ui.domain` inside
     `contents[]` (not resource config).
   - Enable Strict CSP mode in MCPJam Inspector or open VS Code
     `Developer: Toggle Developer Tools` to spot blocked origins.
3. **GitHub Copilot host layer:**
   - Check `MCP: List Servers` → `Show Output` for startup/transport logs, verify
     Workspace Trust (Restricted Mode blocks workspace `.mcp.json`), and check
     Copilot Business/Enterprise MCP policy if tools are missing.
   - If Copilot routes prompts to built-in tools (such as `#todos`) instead of
     yours, tighten server `instructions` and tool descriptions; if user edits
     in confirmation dialogs break handlers, verify Zod + `isError` responses.
4. **Google Antigravity host layer:**
   - Check `_meta["com.google.antigravity/ui"].entrypoints`: `file` must be on
     its own separate tool; `aux_pane`/`global` manual launches pass `{}`;
     `global` omits `message` and `updateModelContext`; inline cards fall back
     to 150 px if no `size-changed` arrives within 100 ms (800 px max).
   - Check 1 MiB persisted `CallToolResult` limit, 4 concurrent `tools/call`
     cap, lack of persistent browser storage, lack of teardown notifications,
     `${PLUGIN_ROOT}`/`${PLUGIN_DATA}` in `mcp_config.json`, and
     `agy plugin validate --json`.
5. **Server transport:**
   - Verify stdio servers log strictly to `stderr` (`stdout` is pure JSON-RPC)
     and remote servers use Streamable HTTP (`/mcp`, not legacy HTTP+SSE).

## Test one hypothesis at a time

- State one evidence-backed hypothesis and test it with the smallest relevant
  check (unit/protocol test, MCPJam fixed-input call, or `agy plugin validate`).
- Fix the cause, not the symptom. If three distinct attempts fail, stop and
  report what was ruled out and what remains unknown.

## Clean up and verify

Remove temporary diagnostics and follow
[verification-before-completion](../verification-before-completion/SKILL.md)
before claiming the fix is complete.
