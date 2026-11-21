# Project guidance

The only file loaded every session; read everything else on demand. Keep this
file under 70 lines.

## Project essentials (fill in)

- **Stack & layout:** <!-- `@modelcontextprotocol/ext-apps` v2 + split `@modelcontextprotocol/*` v2 SDKs, Vite + `vite-plugin-singlefile` (`ui/` or `src/`), MCP server (`server.ts` / `main.ts`), optional Antigravity plugin (`plugin.json`, `mcp_config.json`), `tests/` -->
- **Commands:** <!-- dev: `npm run dev` | build: `npm run build` | test: `npm test` | plugin check: `agy plugin validate` -->
- **Do not edit:** <!-- built single-file `dist/` HTML bundles, `node_modules/`, `~/.gemini/config/plugins/` copies directly, or unrelated repos -->

## When to read what

- Small task (one file, question, quick fix): read nothing extra.
- Substantial work: consult `find-skills` (if installed) once before planning.
- Multi-file or cross-session work: read `#current-handoff` in
  [docs/ai/handoff.md](docs/ai/handoff.md) and [docs/ai/todo.md](docs/ai/todo.md),
  then verify against Git and source.
- Tool, `ui://` view, entrypoint, CSP, or packaging work: read
  [docs/app/spec.md](docs/app/spec.md).
- Large docs: locate the relevant heading and read only that section.
- Procedures (MCP App create/migrate/convert, coordination, debugging,
  verification, memory, canvas, or `setup-clarifydoc`): load from `.agents/skills/`.
- Explicit user direction overrides any standing rule here.

## Rules

- **Truthful.** Run checks before claiming success. Separate unit/protocol
  passes, `basic-host` checks, `agy plugin validate`, and live Antigravity App
  UI verification. Never record secrets, PII, or user paths.
- **MCP App invariants.** Register all `App` handlers before `app.connect()`,
  set `_meta.ui.csp` / `_meta.ui.domain` inside `contents[]`, return plain-text
  `content` for non-UI hosts, and persist edits eagerly (no teardown event).
- **Portable packaging.** Keep contributor scaffolding (root `AGENTS.md`,
  `.agents/`, `.github/`, `scripts/check-context-size.ps1`) separate from
  shipped plugin files; use `${PLUGIN_ROOT}` / `${PLUGIN_DATA}` in
  `mcp_config.json` (never hardcoded paths or duplicated in `env`).
- **Branches & commits.** Develop on `feature/<topic>`, merge into `dev` (then
  delete the merged local branch with `git branch -d`), and PR `dev` into
  `main`. Commit verified batches (main agent / coordinator only; subagents do
  not commit). Do not push or rewrite history unless asked.
- **Ask first.** Stop and ask before changing IDE, user, account, or billing
  settings; modifying `~/.gemini/config/plugins/` outside approved local tests;
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
