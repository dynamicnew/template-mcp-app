---
name: coordinate-task
description: Lead a multi-step change from the main agent, delegate bounded work to subagents (research, implementer, reviewer, handoff), and verify the result. Use for tasks spanning multiple files or independent workstreams.
---

# Coordinate a multi-step change

> **Platform note:** In **GitHub Copilot**, use the `Coordinator` agent
> ([.github/agents/coordinator.agent.md](../../../.github/agents/coordinator.agent.md),
> delegating to `Explorer`, `Implementer`, `Reviewer`, and `Project Handoff`).
> The `invoke_subagent`/`manage_subagents` steps and
> `flash_lite`/`flash`/`inherit`/`pro` tiers below apply to **Google
> Antigravity**'s main agent.

Follow [AGENTS.md](../../../AGENTS.md) and [docs/ai/cost.md](../../../docs/ai/cost.md).
Run coordination from the main agent (which owns `invoke_subagent` and
`manage_subagents`) and continue until the outcome is verified or a real blocker
prevents safe progress.

1. Verify the working directory, branch, and git status before delegating, and
   pass them to subagents when relevant.
2. Run zero to three concurrent subagents. Delegate only bounded, independent
   work with exclusive file ownership and a clear stopping point.
3. Match each subagent to its role, read-list, and `invoke_subagent` model tier:
   - **Explore:** built-in `research` (read-only sandbox) on `flash_lite` or
     `flash`, with [docs/ai/read-lists/explore.md](../../../docs/ai/read-lists/explore.md).
   - **Implement:** `implementer` on `inherit`, with
     [docs/ai/read-lists/implement.md](../../../docs/ai/read-lists/implement.md).
   - **Review:** `reviewer` on `flash` or `pro`, with
     [docs/ai/read-lists/review.md](../../../docs/ai/read-lists/review.md).
   - **Handoff:** `handoff` on `inherit` or `flash` at milestones.
4. Subagents report back to the main agent only. Verify their diffs and check
   results yourself before accepting them and before committing.
5. If a needed tool is unavailable, investigate alternatives and ask the user to
   enable or install it rather than abandoning the task. Keep the active task
   checklist synchronized as milestones finish.
6. Keep the work clean: no temporary files, dead code, or unrelated refactors.
7. Update `docs/ai/handoff.md` at milestones. Ask before changing settings,
   deploying, or taking destructive actions.
8. End with one short report: files changed, checks with exact results, and what
   is unverified.
