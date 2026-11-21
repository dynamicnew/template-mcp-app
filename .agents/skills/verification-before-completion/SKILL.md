---
name: verification-before-completion
description: Run targeted unit/protocol tests, single-file UI build/CSP checks, basic-host or Playwright checks, and agy plugin validate before claiming an MCP App change is done, separating harness checks from live Antigravity App UI verification.
---

# Verification before completion

Evidence before claims. A passing unit test, Vite build, or `basic-host` run is
evidence about that layer — not proof that `agy plugin validate`, local plugin
install, OAuth sign-in, or live Antigravity App `inline` / `aux_pane` /
`global` / `file` rendering works.

## Match evidence to the claim

Before editing, state the expected behavior and smallest relevant checks. Never
report unrun commands as passed, and label mocked or harness results plainly.

| Claim | Required evidence |
|---|---|
| Typecheck / unit / protocol tests passed | Exact local or Docker command with pass/fail/skip counts (including Zod input validation, error handling, and plain-text `content` fallback assertions) |
| Single-file `ui://` bundle valid | Fresh Vite `vite-plugin-singlefile` build output, byte size (`<= 10 MiB`), MIME type `text/html;profile=mcp-app`, and verified `_meta.ui.csp` placement inside `contents[]` |
| Local UI harness verified | Fresh `basic-host` (`SERVERS='["http://localhost:3001/mcp"]'`) and/or Playwright run covering tool round-trip, Light/Dark themes, narrow viewport, and zero console/page errors |
| Antigravity plugin packaging valid | Fresh `agy plugin validate --strict <path>` (clean bundle without root `AGENTS.md` or other-tool files) or `agy plugin validate --json <path> \| jq` filtering only expected contributor/other-tool warnings, exiting `0` |
| Local install works | Fresh `agy plugin install <path>` and `agy plugin list --json` showing expected MCP server and component counts |
| Live Antigravity App UI verified | Explicit manual/live verification in the Antigravity App for each declared entrypoint (`inline`, `aux_pane`, `global`, `file`), reported separately from `basic-host` |
| Docs / scaffolding verified | Relative link check, `git diff --check`, and `scripts/check-context-size.ps1` |

## Check security and hygiene alongside assertions

- Confirm no secrets, OAuth client secrets, or absolute user paths appear in
  source, `plugin.json`, `mcp_config.json`, or docs.
- Confirm untrusted text renders via `textContent` (never raw `innerHTML`),
  server-side validation guards `visibility: ["app"]` and `file` tools (CLI
  ignores `_meta.ui.visibility`), and `node_modules/`, `dist/`, and
  `.plugin-data/` remain untracked.

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
