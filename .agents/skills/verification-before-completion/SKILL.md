---
name: verification-before-completion
description: Run targeted unit/protocol tests, single-file UI/CSP checks, MCPJam or basic-host emulation, and Copilot/Antigravity host checks before claiming an MCP App change is done, keeping evidence tiers strictly separate.
---

# Verification before completion

Evidence before claims. A passing unit test, Vite build, or emulated harness run
is evidence about that layer — not proof that VS Code GitHub Copilot Chat,
`agy plugin validate`, local plugin install, OAuth sign-in, or live Antigravity
App `inline` / `aux_pane` / `global` / `file` rendering works.

## Match evidence to the claim

Before editing, state the expected behavior and smallest relevant checks. Never
report unrun commands as passed, and label mocked or emulated results plainly.

| Claim | Required evidence |
|---|---|
| Layer 1 — Unit & protocol tests passed | Exact local or Docker command with pass/fail/skip counts covering valid inputs, invalid schemas, missing IDs, empty/large results, and plain-text `content` fallback |
| Layer 2 — Single-file `ui://` bundle valid | Fresh Vite `vite-plugin-singlefile` build output, byte size (`<= 10 MiB`), MIME type `text/html;profile=mcp-app`, and verified `_meta.ui.csp` inside `contents[]` |
| Layer 3 — Emulated host harness verified | Fresh `MCPJam Inspector` (Strict CSP mode, JSON-RPC trace, fixed-input calls) and/or `basic-host` / Playwright run covering Light/Dark themes, narrow viewport (`<= 768 px`), and zero console/page errors |
| Layer 4a — VS Code GitHub Copilot verified | Live check in VS Code Copilot Chat agent mode: tool selection alongside built-in tools, `readOnlyHint` auto-run vs editable-argument confirmation dialog, `Show Output`, and DevTools console |
| Layer 4b — Antigravity packaging & App verified | Fresh `agy plugin validate` (`--strict` on clean bundle or `--json \| jq` filtering expected contributor/other-tool warnings), `agy plugin install` + `agy plugin list --json`, and live Antigravity App check per declared entrypoint (`inline`, `aux_pane`, `global`, `file`) |
| Release QA & security verified | Completed [mcp-release-qa](../mcp-release-qa/SKILL.md) and [mcp-implementation-security-review](../mcp-implementation-security-review/SKILL.md) evidence |
| Docs / scaffolding verified | Relative link check, `git diff --check`, and `scripts/check-context-size.ps1` |

## Check security and hygiene alongside assertions

- Confirm no secrets, OAuth client secrets, or absolute user paths appear in
  source, `.mcp.json`, `plugin.json`, `mcp_config.json`, or docs.
- Confirm untrusted text renders via `textContent` (never raw `innerHTML`),
  server-side validation guards `visibility: ["app"]` and `file` tools, and
  `node_modules/`, `dist/`, and `.plugin-data/` remain untracked.

## Verify, clean up, and report

1. Run required checks after the final meaningful edit (rebuilding Docker
   images first when running in Docker).
2. Read full outputs, exit codes, pass/fail/skip counts, bundle sizes, and
   every `agy plugin validate` finding.
3. For delegated work, verify subagent diffs and command outputs yourself
   before accepting or committing on the feature branch.
4. Remove temporary files, scratch fixtures, or debug logs.
5. Report exact commands, counts, and unverified limits plainly. If runtime
   evidence contradicts green checks, switch to
   [systematic-debugging](../systematic-debugging/SKILL.md).
