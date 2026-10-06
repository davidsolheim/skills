---
name: implement
description: >-
  Run the full implement-review-fix loop using implementer and reviewer personas.
  Supports effort-based multi-reviewer scaling (2–7 reviewers) with automatic
  specialization selection. Includes memory-based feedback loop that learns from
  past review patterns. Loops until all reviewers find 0 issues of any severity.
  Edits on the shared checkout use wcp.
when-to-use: Use when asked to "implement", "build", "add feature", "fix bug", or "/implement".
argument-hint: "[--effort N] <description of what to implement>"
disable-model-invocation: true
---

# Implement

Read and follow `~/.grok/bundled/skills/implement/SKILL.md` in full. That file is the procedure.

## Occupancy (wcp)

This loop edits the shared checkout. Inject the player turn into every implementer prompt, including fix rounds. Authority: [`../docs/wcp.md`](../docs/wcp.md) and skill `water-cooler-protocol`.

- Move the issue you take to `in-progress/` and list the paths you write in `files`
- Read other `in-progress/` issues and work around their paths
- Move the issue to `done/` when the writing is finished
- Do not commit while any issue is in `in-progress/`
- Do not reset, checkout, or stash another agent's work
