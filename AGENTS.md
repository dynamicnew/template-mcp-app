# Project guidance

The only file loaded every session; read everything else on demand. Keep this
file under 70 lines.

## Project essentials (fill in)

- **Stack & layout:** <!-- `@modelcontextprotocol/ext-apps` v2 + split `@modelcontextprotocol/*` v2 SDKs, Vite + `vite-plugin-singlefile` (`ui/` or `src/`), MCP server (`server.ts`/`main.ts`), `.mcp.json` (Copilot/portable) + optional Antigravity plugin (`plugin.json`, `mcp_config.json`), `tests/` -->
- **Commands:** <!-- dev: `npm run dev` | build: `npm run build` | test: `npm test` | inspect: `npx @mcpjam/inspector@latest` | Antigravity: `agy plugin validate` -->
- **Do not edit:** <!-- built `dist/` HTML bundles, `node_modules/`, `~/.gemini/config/plugins/` copies directly, or unrelated repos -->

## When to read what

- Small task (one file, question, quick fix): read nothing extra.
- Substantial work: consult `find-skills` (if installed) once before planning.
- Multi-file or cross-session work: read `#current-handoff` in
  [docs/ai/handoff.md](docs/ai/handoff.md) and [docs/ai/todo.md](docs/ai/todo.md),
  then verify against Git and source.
- Tool, `ui://` view, host matrix, CSP, or packaging work: read
  [docs/app/spec.md](docs/app/spec.md).
- Large docs: locate the relevant heading and read only that section.
- Procedures (MCP App create/migrate/convert, QA, security review, coordination,
  debugging, verification, memory, canvas, `setup-clarifydoc`): `.agents/skills/`.
- Explicit user direction overrides any standing rule here.

## Rules

- **Truthful.** Run checks before claiming success. Separate unit/protocol
  passes, `MCPJam`/`basic-host` emulation, `agy plugin validate`, VS Code
  Copilot checks, and live Antigravity App checks. Never record secrets or PII.
- **MCP App invariants.** Register `App` handlers before `app.connect()`, set
  `_meta.ui.csp`/`_meta.ui.domain` inside `contents[]`, return concise plain-text
  `content` for non-UI hosts, and persist edits eagerly (no teardown event).
- **Dual-host packaging.** Keep contributor scaffolding (`AGENTS.md`, `.agents/`,
  `.github/`, `scripts/check-context-size.ps1`) separate from shipped files; use
  `${PLUGIN_ROOT}`/`${PLUGIN_DATA}` in Antigravity `mcp_config.json` and
  `.mcp.json` (`"mcpServers"`) for Copilot/portable hosts.
- **Branches & commits.** Develop on `feature/<topic>`, merge into `dev` (then
  delete the merged branch with `git branch -d`), and stop at `dev`; only PR
  `dev` into `main` when asked to release. Commit verified batches (main agent /
  coordinator only; subagents never commit). Never push/rewrite unless asked.
- **Ask first.** Stop and ask before changing IDE, user, account, or billing
  settings; modifying `~/.gemini/` or `~/.copilot/` outside approved tests;
  accessing signed-in accounts; publishing; or destructive actions.
- **Cost & communication.** Read [docs/ai/cost.md](docs/ai/cost.md) for model
  tiers and delegation. Be terse; no progress narration; end with one short
  report (files changed, exact checks, unverified).
- **Memory.** Record verified lessons with `project-remember`. Keep shared
  decisions in [docs/ai/decisions.md](docs/ai/decisions.md).

## Subagents

Give each subagent only its bounded task, exclusive file ownership and read-list
from [docs/ai/read-lists/](docs/ai/read-lists/) (`research`/`Explorer` ->
`explore.md`, `implementer`/`Implementer` -> `implement.md`,
`reviewer`/`Reviewer` -> `review.md`). `handoff`/`Project Handoff` reads
`docs/ai/handoff.md` and `docs/ai/todo.md` directly.

## Map

| Need | Where |
|---|---|
| State, TODOs, decisions, lessons, cost | [docs/ai/](docs/ai/) (`handoff.md`, `todo.md`, `decisions.md`, `memory.md`, `cost.md`) |
| App spec & optional codebase maps | [docs/app/spec.md](docs/app/spec.md), [docs/codebase/README.md](docs/codebase/README.md) |
| Conditional rules (Antigravity / Copilot) | [.agents/rules/](.agents/rules/), [.github/instructions/](.github/instructions/) |
| Subagent roles (Antigravity / Copilot) | [.agents/agents/](.agents/agents/), [.github/agents/](.github/agents/) |
| Shared skills (Antigravity + Copilot) | [.agents/skills/](.agents/skills/) |
