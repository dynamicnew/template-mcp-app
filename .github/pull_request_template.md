## Summary

- What changed and why:
- Source branch (`dev` from `feature/<topic>`, targeting `main`):

## Verification (Cross-Host Testing Pyramid)

- [ ] **Layer 1 — Unit & protocol tests** (local or Docker): <!-- exact command + pass/fail counts (valid, invalid, empty, large-result, plain-text fallback) -->
- [ ] **Layer 2 — Single-file `ui://` build & CSP** (`<= 10 MiB`, `text/html;profile=mcp-app`, `_meta.ui.csp` inside `contents[]`, Light/Dark + narrow viewport): <!-- bundle path + byte size -->
- [ ] **Layer 3 — Emulated host harness** (`MCPJam Inspector` Strict CSP / JSON-RPC trace and/or `basic-host` / Playwright): <!-- exact command or why skipped -->
- [ ] **Layer 4a — VS Code GitHub Copilot Chat** (tool disambiguation vs built-in tools, `readOnlyHint` vs editable-argument confirmation dialog, `Show Output` & DevTools): <!-- result or N/A -->
- [ ] **Layer 4b — Google Antigravity** (`agy plugin validate`, local `agy plugin install`, declared `inline`/`aux_pane`/`global`/`file` entrypoints): <!-- exit code / findings or N/A -->
- [ ] **Release QA & Security** (`mcp-release-qa` / `mcp-implementation-security-review` if shipping): <!-- summary or N/A -->
- [ ] **Hygiene** (no secrets, no hardcoded user paths, LF endings, no tracked `node_modules/` or `dist/`)

## Unverified / Limits

- <!-- e.g., live VS Code Copilot Chat rendering, live Antigravity App aux_pane/global/file rendering, OAuth browser sign-in -->

## Documentation & Handoff

- [ ] Updated `docs/ai/handoff.md` (and `docs/ai/decisions.md` / `docs/app/spec.md` if applicable)
- [ ] Kept internal contributor notes out of user-facing README and tool descriptions
