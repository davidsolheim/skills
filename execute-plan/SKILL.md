---
name: execute-plan
description: >-
  Execute a PR Plan DAG from a design document. Parses the plan, topologically
  sorts it, implements PRs in parallel using worktree-isolated subagents, runs
  mandatory orchestrator-level review, and assembles either a Graphite PR stack
  or a plain-git branch stack depending on tool availability. Worktrees do not
  share one wcp board. Orchestrator edits on the shared
  checkout do.
when-to-use: Use when asked to "execute plan", "run the plan", "implement the design", or "/execute-plan".
argument-hint: "<design-doc-path> [--effort N] [--concurrency N] [--dry-run] [--resume <PLAN_ID>] [--instructions \"...\"] [--no-graphite] [--auto-pr]"
disable-model-invocation: true
---

# Execute plan

Read and follow `~/.grok/bundled/skills/execute-plan/SKILL.md` in full. That file is the procedure.

## Occupancy (wcp)

Implementers each have their own worktree. Do not put those worktrees on the shared checkout's wcp board. Authority: [`../docs/wcp.md`](../docs/wcp.md).

When the orchestrator edits the shared checkout, follow [`../docs/wcp.md`](../docs/wcp.md). Read `in-progress/` and work around those paths. Do not commit while an issue is in `in-progress/`.

Queue work on one shared `dev` branch is `/solve`, not this skill.
