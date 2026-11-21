---
applyTo: "server/**,src/**,main.ts,server.ts,package.json,.mcp.json*,.agents/mcp_config.json*,mcp_config.json,.vscode/mcp.json*"
description: Cross-host MCP server transport (stdio and Streamable HTTP), portable config files, authentication, security baseline, and tool contract rules.
---

- **Dual transport support:**
  - **Local stdio:** Default to `tsx` in development and `"command": "node"` in
    shipped configs so the server runs cross-platform (and falls back to
    Antigravity's bundled Node.js runtime). Never write logs or banners to
    `stdout` on stdio servers (use `stderr`); `stdout` carries JSON-RPC frames
    exclusively.
  - **Remote Streamable HTTP:** Serve over Streamable HTTP (conventionally at
    `/mcp`); legacy HTTP+SSE and `httpUrl` are unsupported.
- **Portable multi-host configuration:**
  - Keep `.mcp.json.example` (`"mcpServers"`, shared by VS Code Copilot &
    Copilot CLI) and `.agents/mcp_config.json.example` (`"mcpServers"`,
    Antigravity workspace) free of secrets and absolute user paths. Note that
    legacy `.vscode/mcp.json` uses `"servers"`.
  - Pin exact reviewed versions for any package-runner references (`npx`, `uvx`)
    in shipped configs instead of bare package names or `@latest`.
- **Auth & credential safety:** Never commit API keys, tokens, or OAuth client
  secrets. Prefer OAuth 2.0 + PKCE for remote servers (supported across VS Code
  Copilot, Antigravity, and M365 Copilot), `"authProviderType":
  "google_credentials"` for Google ADC in Antigravity, or user-prompted
  credentials (`mcpServers.<server>.variables` in Antigravity plugins; `inputs`
  / env vars in VS Code). Remember Antigravity plugins declaring `variables`
  install disabled via CLI until configured.
- **Tool quality & security baseline:** Validate all tool arguments with strict
  Zod schemas, set accurate annotations (`title`, `readOnlyHint`,
  `destructiveHint`, `idempotentHint`), return actionable `isError: true`
  responses, enforce server-side authorization on `visibility: ["app"]` and
  `file` tools, and run `mcp-implementation-security-review` before shipping
  network-exposed or mutation-capable servers.
