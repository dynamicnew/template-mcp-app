---
name: systematic-debugging
description: Investigate MCP App failures—lifecycle handler races, single-file ui:// bundling, CSP/CORS blocks, Antigravity entrypoints (inline/aux_pane/global/file), stdio/HTTP transport errors, and plugin packaging issues—before changing code.
---

# Systematic debugging

Find the root cause of the observed MCP App failure, not just a workaround.
Follow [AGENTS.md](../../../AGENTS.md) and never loosen CSP/schemas, swallow
errors, or commit hardcoded credentials/paths.

## Establish the actual host surface and build

- Read `#current-handoff` in [docs/ai/handoff.md](../../../docs/ai/handoff.md)
  and relevant sections of [docs/app/spec.md](../../../docs/app/spec.md).
- Confirm which surface failed: unit/protocol test, Vite single-file build,
  `basic-host` reference harness, `agy plugin validate`, Antigravity CLI
  (`agy`), Antigravity App (`inline`, `aux_pane`, `global`, `file`), or another
  MCP host.
- Confirm you are testing the fresh build, not a stale copy: rebuild the
  single-file HTML bundle, close and reopen `ui://` views for HTML edits,
  restart local stdio servers in Customizations after server code edits, and
  reinstall after manifest edits.

## Collect evidence by layer

1. **SDK & lifecycle (`@modelcontextprotocol/ext-apps`):**
   - Check that all handlers (`ontoolinput`, `ontoolinputpartial`,
     `ontoolresult`, `onhostcontextchanged`, `onteardown`) are registered
     BEFORE `app.connect()`.
   - Verify split MCP SDK v2 packages (`@modelcontextprotocol/client`, `server`,
     `node`, `express`) are used consistently with zero v1 SDK conflicts.
   - Use `app.sendLog()` to surface iframe errors to the host.
2. **`ui://` bundle & CSP/CORS:**
   - Verify `registerAppResource` returns `text/html;profile=mcp-app`
     (`RESOURCE_MIME_TYPE`), the single-file bundle is `<= 10 MiB`, and
     `_meta.ui.csp` / `_meta.ui.domain` sit inside `contents[]` (not resource
     config).
   - Inspect console/CSP violations for blocked origins or missing
     `connectDomains` / `resourceDomains`.
3. **Antigravity entrypoints & host limits:**
   - Check `_meta["com.google.antigravity/ui"].entrypoints`: `file` must be on
     its own separate tool; `aux_pane`/`global` manual launches pass `{}`;
     `global` omits `message` and `updateModelContext`; inline cards fall back
     to 150 px if no `size-changed` arrives within 100 ms (800 px max).
   - Check 1 MiB persisted `CallToolResult` limit, 4 concurrent `tools/call`
     cap, lack of persistent browser storage, and lack of
     `ui/resource-teardown` on close.
4. **Server transport & plugin packaging:**
   - Verify stdio servers log only to `stderr` (`stdout` is pure JSON-RPC),
     remote servers use Streamable HTTP (`serverUrl`/`url`), `mcp_config.json`
     uses `${PLUGIN_ROOT}` / `${PLUGIN_DATA}` without redeclaring them in `env`,
     and `agy plugin validate --json` reports zero Antigravity errors.

## Test one hypothesis at a time

- State one evidence-backed hypothesis and test it with the smallest relevant
  check (unit/protocol test, `basic-host`, or `agy plugin validate`).
- Fix the cause, not the symptom. If three distinct attempts fail, stop and
  report what was ruled out and what remains unknown.

## Clean up and verify

Remove temporary diagnostics and follow
[verification-before-completion](../verification-before-completion/SKILL.md)
before claiming the fix is complete.
