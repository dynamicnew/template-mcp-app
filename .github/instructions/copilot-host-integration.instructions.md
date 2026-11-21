---
applyTo: "src/**,server/**,ui/**,web/**,.mcp.json*,.vscode/mcp.json*,package.json"
description: GitHub Copilot (VS Code Chat, Copilot CLI, and M365 Copilot) MCP App integration rules: inline layout, tool disambiguation, approval UX, configuration, and debugging.
---

- **VS Code Copilot Chat rendering & capabilities:**
  - Copilot renders MCP Apps inline in agent mode (design layouts to fit cleanly
    within ~768 px wide × 400 px tall with wrapping and zero horizontal overflow
    on narrower panes). Check `hostContext.availableDisplayModes` before
    exposing fullscreen toggles.
  - `app.sendMessage()` (`ui/message`) pre-fills the chat input rather than
    auto-sending; `app.updateModelContext()` attaches context for later turns.
  - Tool-result media/images are visible to the model but not placed into the
    app surface; render images inside the UI bundle (or allowlist their origin
    in `resourceDomains`).
  - Re-rendering a view resets client-only UI state; persist durable selections
    or edits on the server or via `app.updateModelContext()`.
- **Tool disambiguation & approval UX:**
  - Copilot agent mode exposes built-in tools (such as `#todos` and workspace
    tools) alongside your MCP server. Provide clear server `instructions` and
    domain-specific tool descriptions so the model does not route user requests
    to built-in Copilot tools.
  - Copilot displays each tool's `title` in chat and auto-runs tools marked
    `readOnlyHint: true` without prompting. All other tools show a confirmation
    dialog where the user can **edit arguments** before execution: validate
    every argument server-side and return actionable `isError: true` messages.
- **Configuration, Workspace Trust & policies:**
  - Prefer workspace-root `.mcp.json` (`"mcpServers"`, shared across teammates
    and Copilot CLI) or `Global > Copilot Global` (`~/.copilot/mcp-config.json`,
    `"mcpServers"`). Legacy `.vscode/mcp.json` uses `"servers"` and is now
    superseded by `.mcp.json` / Copilot Global in VS Code's guided flow.
  - Workspace `.mcp.json` / `.vscode/mcp.json` inherit **Workspace Trust** (they
    cannot start in Restricted Mode). Never commit secrets; use `inputs`, env
    vars, or MCP OAuth. On Copilot Business/Enterprise, an admin must enable MCP
    servers in Copilot policy.
- **Debugging & Copilot CLI fallback:**
  - Inspect server logs via `MCP: List Servers` → `Show Output` and iframe
    CSP/JS errors via `Developer: Toggle Developer Tools`.
  - GitHub Copilot CLI runs MCP tools without mounting `ui://` iframes: keep
    plain-text `content` complete and self-explanatory. *(Copilot CLI SDK canvas
    extensions under `.github/extensions/` are a separate API — see
    `handoff-canvas`.)*
