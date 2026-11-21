---
name: setup-clarifydoc
description: Detect whether a project supports ClarifyDoc when adopting a template, initializing a repository, or preparing an initial commit, and scaffold only the needed ClarifyDoc MCP guidance, dual-platform rules, workflow skills, and ignore rules on demand.
---

# Bootstrap ClarifyDoc support (on demand)

Run this procedure when a template is applied, a repository is initialized or
committed for the first time, or the user asks to enable ClarifyDoc. **If the
project does not support or use ClarifyDoc, make no changes and create no
ClarifyDoc files.**

## 1. Detect whether ClarifyDoc applies

Check for any of these signals:
- `clarifydoc_*` MCP tools are available in the active session (confirm with
  one read-only `clarifydoc_status` call if present).
- A `.clarifydoc/` folder or existing ClarifyDoc MCP entry already exists in
  the workspace or user/editor config.
- The user explicitly requests ClarifyDoc setup.

If none apply, stop — do not pre-create unused ClarifyDoc assets.

## 2. Configure MCP safely (no secrets or machine-local paths)

- Reuse a single already-configured global/editor ClarifyDoc MCP provider when
  available; never register duplicate providers (for example `clarifydoc` and
  `ClarifyDoc`), which double the 23 tools into 46.
- Remember that the ClarifyDoc MCP server's target directory is fixed at server
  startup (`clarifydoc mcp <path>`); neither `clarifydoc_status` nor
  `clarifydoc_analyze` takes a `path` argument.
- Never commit machine-local absolute binary paths (such as editor
  `globalStorage` paths) or secrets into tracked `.mcp.json`,
  `.vscode/mcp.json`, or `.agents/mcp_config.json`.
- If workspace MCP setup is needed, ask the user before writing active MCP
  config or guide them to configure user-level MCP (Antigravity
  `~/.gemini/config/mcp_config.json`, or VS Code's **ClarifyDoc: Set Up MCP
  for an AI Agent...** / **Copy MCP Server Config**). Remember `.mcp.json` and
  `.agents/mcp_config.json` use `"mcpServers"` while `.vscode/mcp.json` uses
  `"servers"`.

## 3. Scaffold dual-platform repository assets tailored to this project

Create only the assets appropriate for the repository's actual stack:

1. **`.gitignore`:** add `.clarifydoc/` so generated analysis artifacts are
   never committed or hand-edited.
2. **Conditional rules (both platforms, keeping `AGENTS.md` <= ~70 lines):**
   - Antigravity: `.agents/rules/clarifydoc.md` (`trigger: model_decision` and
     non-empty `description:` in YAML frontmatter).
   - GitHub Copilot: `.github/instructions/clarifydoc.instructions.md`
     (`applyTo` covering the project's source paths and non-empty `description:`
     in YAML frontmatter).
   - Rules to include:
     - Call `clarifydoc_status` first through one provider and verify the server
       is bound to the current repository root. If `clarifydoc_status` reports a
       home-directory error or a different project path, **stop and never call
       `clarifydoc_analyze`** (`clarifydoc_analyze` will scan a home directory
       instead of rejecting it; cancel immediately with `clarifydoc_analyze(cancel=true)`
       if ever triggered on the wrong root) and instruct the user to start the
       server for this repo (`clarifydoc mcp <repo-path>`).
     - Report missing/stale analysis rather than silently refreshing.
     - Use `clarifydoc_explain`, `clarifydoc_blast_radius`, or `clarifydoc_impact`
       before modifying routes, screens, or symbols; use `clarifydoc_search`
       before writing new helpers.
     - Verify any row marked `inferred` or `ambiguous` — and any dynamic entry
       points or static "unreachable" candidates — directly in source before
       editing or deleting code. Distinguish indexed handlers (`clarifydoc_file`)
       from recognized routes (`clarifydoc_routes`): file-based serverless
       handlers can be indexed while `clarifydoc_routes` reports `0` routes and
       `clarifydoc_screens` labels their callers as unmatched.
     - Require user confirmation before running write/cache operations
       (`clarifydoc_analyze` or `clarifydoc_diff` baseline writes). If a
       read-only MCP server (for example, started with `--no-auto-analyze`)
       rejects `clarifydoc_analyze`, fall back (after user approval) to
       `clarifydoc analyze <repo-path>` if `clarifydoc` is on `PATH` (or via its
       installed launcher), or direct the user to VS Code's **ClarifyDoc: Analyze
       Workspace**; note that LSP enhancement needs Docker or **ClarifyDoc:
       Resolve Calls with the Editor's Language Servers**.
     - Never hand-edit `.clarifydoc/`, and review diffs if a ClarifyDoc guidance
       installer tries to append a large boilerplate block to `AGENTS.md`.
3. **Shared portable workflow skills under `.agents/skills/`** (single copy
   loaded by both Antigravity and GitHub Copilot; do not duplicate into
   `.github/skills/`; match each folder name in frontmatter `name:` and use real
   example paths from this repo):
   - `clarifydoc-review/SKILL.md` — analysis-only bounded query workflow
     (`status`, smallest query set, source verification, no silent refresh).
   - `clarifydoc-before-editing/SKILL.md` — pre-edit map (`file`,
     `blast_radius`, `explain`, `trace` callers, `context_docs`, `impact`).
   - `clarifydoc-before-writing/SKILL.md` — reuse & installed-API check
     (`search`, `search body=true`, `explain`, `extract`).
   - `clarifydoc-refactor/SKILL.md` — 4-phase map -> plan -> edit -> verify
     (`blast_radius`, `trace`, `health category="layers"`,
     `architecture view="cycles"`, `context_docs`, then post-edit verification).
   - `clarifydoc-review-branch/SKILL.md` — branch change review (`impact`,
     `contract`, `health`).
   - *(Include only when the repo has UI screens + API routes)*
     `clarifydoc-explain-screen/SKILL.md` (`screens`, `routes`, `data_flows`)
     and `clarifydoc-trace-feature/SKILL.md` (`search`, `screens`, `routes`,
     `data_flows`, `trace`).
4. **`AGENTS.md` pointer (optional):** keep `AGENTS.md` under ~70 lines; at
   most add ClarifyDoc to the existing `Map` table rather than pasting a tool
   catalog into always-on context.

## 4. Verify before committing

1. Run `pwsh -NoProfile -File scripts/check-context-size.ps1 -FailOnWarning`
   and confirm exit `0`, zero duplicate skills across `.agents/skills/` and
   `.github/skills/`, `AGENTS.md` <= ~70 lines, and valid active workspace MCP
   configs with zero secrets or machine-local paths (and no hyphenated
   `.agents/mcp-config.json`).
2. Confirm relative links, frontmatter, and `.gitignore` coverage for
   `.clarifydoc/`.
3. Check `clarifydoc_status` first: only if it confirms the MCP server is bound
   to this repository root (not a home directory or another repo), ask before
   running an initial `clarifydoc_analyze`, then re-check `clarifydoc_status`.
   If the MCP server is read-only and rejects `clarifydoc_analyze`, ask before
   running `clarifydoc analyze <repo-path>` via the CLI (or **ClarifyDoc:
   Analyze Workspace**) and report CLI vs MCP evidence separately. If bound to
   a different root, do not call `clarifydoc_analyze`; report the root mismatch
   and instruct the user to launch `clarifydoc mcp <repo-path>`.
4. Note that newly created skills may require a fresh session to verify live
   discovery/activation.
