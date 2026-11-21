## Summary

- What changed and why:
- Source branch (`dev` from `feature/<topic>`, targeting `main`):

## Verification

- [ ] Typecheck / unit / MCP protocol tests (local or Docker): <!-- exact command + pass/fail counts -->
- [ ] Single-file `ui://` HTML build verified (`<= 10 MiB`, no unexpected external scripts/fonts, CSP `_meta.ui.csp` set inside `contents[]`): <!-- bundle path + byte size -->
- [ ] Plain-text `content` fallback verified for non-UI hosts (Antigravity CLI / IDE)
- [ ] Local UI harness (`basic-host` or Playwright) checked for Light/Dark theme, narrow viewport, and console errors: <!-- exact command or why skipped -->
- [ ] Antigravity plugin packaging checked via `agy plugin validate` (`--strict` on clean bundle or `--json | jq` filtering expected contributor/other-tool warnings) and local `agy plugin install` (if packaged as a plugin): <!-- exit code / findings or N/A -->
- [ ] Hygiene checked (no secrets, no hardcoded user paths, LF endings, no tracked `node_modules/` or `dist/`)

## Unverified / Limits

- <!-- e.g., live Antigravity App inline / aux_pane / global / file viewer rendering, OAuth browser sign-in -->

## Documentation & Handoff

- [ ] Updated `docs/ai/handoff.md` (and `docs/ai/decisions.md` / `docs/app/spec.md` if applicable)
- [ ] Kept internal contributor notes out of user-facing README and tool descriptions
