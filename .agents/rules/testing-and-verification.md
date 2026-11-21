---
trigger: model_decision
description: Hybrid testing, single-file bundle checks, basic-host/Playwright UI verification, and agy plugin validate rules for Antigravity MCP Apps.
---

- **Hybrid test model:** Run deterministic, offline unit, Zod schema, and MCP
  protocol tests locally or in Docker as configured for the project, using
  disposable fixtures only (never mutating real user accounts or live
  `~/.gemini/` state).
- **Single-file UI & protocol verification:**
  - Verify the single-file `ui://` HTML bundle builds cleanly, stays `<= 10 MiB`,
    serves MIME type `text/html;profile=mcp-app`, and declares `_meta.ui.csp`
    inside `contents[]` without unintended external script/font fetches.
  - Test UI rendering and tool round-trips locally with the MCP Apps
    `basic-host` reference harness (`SERVERS='["http://localhost:3001/mcp"]'`)
    and/or Playwright tests covering Light/Dark themes, narrow viewports,
    `prefers-reduced-motion`, and zero console/page errors.
- **Antigravity packaging & iteration checks:**
  - Clean plugin bundle (no root `AGENTS.md` or other-tool files): require
    `agy plugin validate --strict <path>` to exit `0` with zero findings.
  - Repo root containing contributor `AGENTS.md` or other-tool files
    (`.mcp.json`, `mcp.json`, shadowed `plugin.json`): run
    `agy plugin validate <path>` or filter only those expected contributor
    warnings with `agy plugin validate --json <path> | jq -e ...` so real
    plugin warnings still fail CI.
  - Qualify local installs with `agy plugin install <path>` and
    `agy plugin list --json` (ask before changing global `~/.gemini/` outside
    approved local tests). Remember `ui://` HTML edits apply on closing and
    reopening the view, whereas local stdio server edits require restarting the
    server in Customizations.
- **Honest evidence tiers:** Report unit/protocol passes, `basic-host` /
  Playwright harness checks, `agy plugin validate`, local CLI installs, and
  live Antigravity App UI rendering (`inline`, `aux_pane`, `global`, `file`)
  separately. Never infer live Antigravity App iframe or OAuth behavior from
  unit tests, `basic-host`, or CLI runs.
- **Repository hygiene:** Never commit `node_modules/`, `dist/`, `.plugin-data/`,
  secrets, or absolute user paths; preserve LF endings (`.gitattributes`).
