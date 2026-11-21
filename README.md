# Agent project template: Antigravity MCP App (Dual-Platform)

A reusable, dual-platform (**Google Antigravity** and **GitHub Copilot**)
agent scaffolding template for building, migrating, and packaging interactive
**MCP Apps** (`io.modelcontextprotocol/ui` + `@modelcontextprotocol/ext-apps`
v2) targeting the Google Antigravity App (inline cards, `aux_pane` side panels,
`global` full-page views, and `file` custom viewers/editors) as well as other
MCP Apps-compatible hosts. Contains no application code, project lore,
machine-specific paths, or secrets.

> **Choosing between templates:** Use this template when your primary focus is
> building, migrating, or converting an interactive MCP App with bundled UI
> views (`@modelcontextprotocol/ext-apps` skills pre-installed). Use
> `antigravity-plugin` when building a broad multi-component Antigravity plugin
> (skills, custom agents, rules, hooks, and optional MCP servers).

## How loading works

Files on disk cost zero tokens until loaded. Both Antigravity and GitHub Copilot
automatically discover root `AGENTS.md` and portable skills under
`.agents/skills/` (the open Agent Skills standard). Keep only `AGENTS.md` at the
root (no `GEMINI.md`). If you use GitHub Copilot Code Review or Visual Studio,
add a one-line `.github/copilot-instructions.md` containing:
`Follow [AGENTS.md](../AGENTS.md).` Do not use `@path` or `@[label](path)`
includes in either file.

| Layer | Loaded when | Platform | Cost |
|---|---|---|---|
| `AGENTS.md` | Every session | Both | Small and fixed (the only always-on file, <= ~70 lines) |
| `.agents/rules/*.md` | `trigger: model_decision` | Antigravity | Only when relevant |
| `.github/instructions/*.instructions.md` | Matching file (`applyTo`) | GitHub Copilot | Only when matching files are touched |
| `.agents/skills/*` | Names/descriptions at startup; `SKILL.md` on use | Both (portable) | Progressive disclosure |
| `.agents/agents/*` / `.github/agents/*` | When invoked / delegated | Platform-native | Only when used |
| `docs/ai/*` & `docs/app/*` | When `AGENTS.md` or rules point to them | Both | Only on demand, by heading |
| `docs/ai/read-lists/*` | When a subagent starts | Both | One small file per subagent role |

> **Contributor vs. shipped separation:** When packaging your MCP App as an
> Antigravity plugin, the plugin loader reads shipped files from `plugin.json`
> (or `.antigravity-plugin/plugin.json`), `mcp_config.json`, `scripts/`, and
> `assets/`. Root `AGENTS.md`, `.agents/`, `.github/`, `.mcp.json`, and
> `scripts/check-context-size.ps1` are contributor-only scaffolding: root
> `AGENTS.md` triggers an expected warning under `agy plugin validate --strict`
> at repo root (filter expected contributor/other-tool warnings with
> `agy plugin validate --json | jq`, or run `--strict` on a clean staged bundle).
>
> **VS Code note:** Keep `.agents/skills` enabled in `chat.agentSkillsLocations`
> so GitHub Copilot loads the single canonical skill tree in `.agents/skills/`
> without duplicating skills into `.github/skills/`.

## Adopt it

1. Click **Use this template** on GitHub (or copy the contents of
   `antigravity-mcp-app/` into your repository root).
2. Fill in `Project essentials` in [AGENTS.md](AGENTS.md) with your UI
   framework, build/test commands, and packaging target (keep `AGENTS.md` under
   ~70 lines).
3. Fill in [docs/app/spec.md](docs/app/spec.md) with your tools, `ui://`
   resources, Antigravity entrypoints (`inline`, `aux_pane`, `global`, `file`),
   CSP allowlists, and auth/state boundaries.
4. Invoke one of the bundled official MCP Apps skills to scaffold or migrate:
   - `create-mcp-app` — build a new MCP App from scratch
   - `add-app-to-server` — add interactive UI views to an existing MCP server
   - `convert-web-app` — turn an existing web app into a hybrid web + MCP App
   - `migrate-oai-app` — migrate an OpenAI Apps SDK app to MCP Apps
5. Update [docs/ai/cost.md](docs/ai/cost.md) with your Antigravity and/or GitHub
   Copilot plan limits and preferred models.
6. If this repository supports ClarifyDoc, run `setup-clarifydoc` during
   adoption or initial setup to generate the ClarifyDoc rule, skills,
   `.gitignore` entry, and MCP guidance on demand (otherwise create none).
7. Run `pwsh -NoProfile -File scripts/check-context-size.ps1 -FailOnWarning` to
   verify always-on budgets, Antigravity rule caps, and skill deduplication.

## Curated skills included

Stored once in `.agents/skills/` for both platforms:

- **Official MCP Apps (`modelcontextprotocol/ext-apps`):** `create-mcp-app`,
  `add-app-to-server`, `convert-web-app`, `migrate-oai-app`.
- **Core workflow:** `coordinate-task` (Antigravity main-agent coordination;
  Copilot uses `.github/agents/coordinator.agent.md`), `project-remember`,
  `systematic-debugging` (MCP Apps lifecycle, CSP/CORS, `ui://` bundling,
  Antigravity entrypoints, and stdio/HTTP server diagnostics),
  `verification-before-completion` (evidence matrix separating unit/protocol
  tests, single-file UI build/size/CSP checks, `basic-host` harness,
  `agy plugin validate`, and live Antigravity App UI checks).
- **On-demand generators:** `generate-handoff-canvas` (recreates a Copilot
  `.github/extensions/<id>/` canvas or Antigravity MCP App side-pane panel),
  `setup-clarifydoc` (detects ClarifyDoc support at template adoption/init and
  scaffolds only the needed ClarifyDoc config, rules, and workflow skills).

## Optional external skills

Heavy vendor skill bundles are deliberately excluded beyond the four official
`ext-apps` skills. If your project wants deeper MCP server or UI craft guides,
review and install only what you need via the open Skills CLI:

- `npx skills add anthropics/skills@mcp-builder` — TypeScript/Python MCP server
  tool design and evaluation guidance
- `npx skills add vercel-labs/agent-skills@web-design-guidelines` — UI and
  accessibility review guidelines
- `npx skills add anthropics/skills@frontend-design` — custom interface design
  polish

## What is deliberately omitted

- **No application code, `package.json`, secrets, or machine-local paths.** Add
  dependencies with your package manager (`npm install`) when scaffolding so
  versions resolve cleanly. Workspace MCP configs live as empty `.example`
  templates (`.agents/mcp_config.json.example` and `.mcp.json.example`); note
  that `.mcp.json`, `.agents/mcp_config.json`, and shipped plugin
  `mcp_config.json` use `"mcpServers"`, while VS Code's `.vscode/mcp.json`
  expects `"servers"`. Never commit literal secrets or user paths.
- **No enabled workspace hooks.** See `.agents/hooks/README.md` and
  `.github/hooks/README.md`.
- **No prebuilt canvas extension or ClarifyDoc skill trees.** Use
  `generate-handoff-canvas` and `setup-clarifydoc` only in repos that need them.

## Layout

```text
AGENTS.md                                  always-loaded dual-platform router (<= ~70 lines)
.agents/
  rules/                                   7 Antigravity conditional rules (model_decision)
  agents/                                  implementer, reviewer, handoff (explore uses built-in research)
  skills/                                  10 shared portable skills (4 official ext-apps + core + generators)
  hooks/README.md                          workspace hook policy (none enabled)
  mcp_config.json.example                  empty workspace MCP server list (Antigravity format)
.github/
  instructions/                            7 Copilot path-scoped instructions (applyTo)
  agents/                                  coordinator, explorer, implementer, reviewer, handoff
  hooks/README.md                          workspace hook policy (none enabled)
  pull_request_template.md                 PR verification & MCP App release checklist
docs/
  ai/                                      handoff.md, handoff-history/, todo.md, decisions.md, memory.md, cost.md, read-lists/
  app/spec.md                              MCP App tools, ui:// resources, entrypoints, CSP & packaging spec
  codebase/README.md                       index for optional on-demand architecture maps
scripts/check-context-size.ps1             contributor-only context-budget & hygiene check
.mcp.json.example                          empty workspace MCP server list (Copilot CLI / portable format)
.gitignore  .gitattributes                 dependency/build/secret hygiene & consistent LF endings
```
