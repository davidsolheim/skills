---
id: "0112"
title: "Publish sanitized /solve skill to davidsolheim/skills"
status: done
priority: high
assignee:
lease_expires:
scope: "Imported from Linear TWV-112. Stay inside that description."
acceptance: "Full solve/ tree (8 files) on public main"
files: []
commit:
reason:
created: "2026-07-31T14:20:05.265Z"
linear_id: "TWV-112"
linear_url: "https://linear.app/teton-web-ventures/issue/TWV-112/publish-sanitized-solve-skill-to-davidsolheimskills"
linear_status: "Done"
linear_status_type: "completed"
linear_team: "TWV"
linear_project: "davidsolheim/skills"
linear_assignee: "David Solheim <david@tetonweb.com>"
linear_labels: []
linear_priority: "High"
linear_parent: ""
linear_cycle: ""
linear_due: ""
linear_updated: "2026-07-31T15:03:13.598Z"
linear_archived: false
notion_page_id: "3e71027c-242b-8167-a94e-f35964dbea0e"
notion_url: "https://app.notion.com/p/3e71027c242b8167a94ef35964dbea0e"
---

## Linear import

- Identifier: TWV-112
- URL: https://linear.app/teton-web-ventures/issue/TWV-112/publish-sanitized-solve-skill-to-davidsolheimskills
- Linear status: Done (completed)
- Queue status: done
- Team: Teton Web Ventures (TWV)
- Project: davidsolheim/skills
- Assignee: David Solheim <david@tetonweb.com>
- Labels: none
- Parent: none
- Priority: High
- Estimate: none
- Cycle: none
- Due: none
- Created: 2026-07-31T14:20:05.265Z
- Updated: 2026-07-31T15:03:13.598Z
- Completed: 2026-07-31T15:03:13.537Z
- Canceled: no
- Archived: no
- Branch: david/twv-112-publish-sanitized-solve-skill-to-davidsolheimskills

Queue status follows Water Cooler Protocol. Todo, In Progress, In Review, Triage, and Backlog are `open` so the import does not take a ticket lease or start a review. Done, Canceled, and Blocked use those folders. `linear_status` is the Linear status at import.

## Description

## Summary

Port the private Grok `/solve` skill (Linear pick → implement loop → merge to local `dev`) into the public [davidsolheim/skills](<https://github.com/davidsolheim/skills>) repo. This is the largest skill package; sanitize all references, custom instructions, and guidance templates so nothing identifies private clients, personal machines, or internal platform product names as required defaults.

## User report

Publish custom Grok skills publicly with no PII and no local project references.

## Current behavior

Source skill (private):

* `solve/SKILL.md`
* `solve/references/git-dev-workflow.md`
* `solve/references/graph.schema.json`
* `solve/references/batch-guidance.md`
* `solve/references/batch-guidance-template.md`
* `solve/references/architecture-guidance-template.md`
* `solve/references/custom-implement-instructions.md`
* `solve/references/fast-mode.md`

**Known non-portable / sensitive patterns:**

* Issue prefix examples `TW-123`, epic `TW-100`
* Preferred project example `Teton Web Platform`
* Absolute skill paths under `~/.grok/skills/solve/...`
* Stack migration examples that may read as org-specific history (ClickHouse/Convex → Neon) — keep only as **generic** multi-issue ordering examples if useful, not as this org’s roadmap
* Any client names or private product names in custom-implement-instructions

## Expected behavior

Public layout:

```text
solve/
  SKILL.md
  references/
    git-dev-workflow.md
    graph.schema.json
    batch-guidance.md
    batch-guidance-template.md
    architecture-guidance-template.md
    custom-implement-instructions.md
    fast-mode.md
```

Preserve:

* Modes: `/solve`, `/solve N`, `/solve all`, `fast`, concurrency flags
* Epic expansion (never implement epic shell)
* Batch guidance + supersession ordering
* Fast worktree orchestrator rules
* Delivery default: merge to **local** `dev` **only** (no push/PR unless user asks)
* Linear claim / Done lifecycle

Make portable:

* Linear resolution from repo docs, not “prefer Teton Web Platform”
* Reference paths relative to skill package (`references/...`) not `~/.grok/skills/solve/...`
* Generic issue ids (`TEAM-123`)

## Suspected root cause / scope

Content port + path/example sanitization across 8 files. Review `custom-implement-instructions.md` carefully — highest risk of private process leakage.

## Code map

| Path | Role |
| -- | -- |
| Private `solve/SKILL.md` | Orchestrator contract |
| Private `solve/references/fast-mode.md` | Parallel worktree protocol |
| Private `solve/references/batch-guidance*.md` | Multi-issue ordering |
| Private `solve/references/custom-implement-instructions.md` | Injected into implement subagents |
| Private `solve/references/git-dev-workflow.md` | Local dev branch rules |
| Public `solve/` | Publish target |

## Implementation notes

1. Copy entire tree.
2. Rewrite path references to skill-relative paths so installs work from any skills root.
3. Replace Teton/TW examples with generic ones.
4. Sanitize custom implement instructions: keep quality bar; remove any org-only tools, private URLs, or client procedures.
5. Keep graph schema if still valid; ensure no embedded private names.
6. Cross-skill mentions (`/implement`, `/prb`) by name only.

## Acceptance criteria

- [ ] Full `solve/` tree (8 files) on public `main`
- [ ] No `Teton Web Platform`, `TW-` as required prefix, `~/.grok/skills/solve` absolute/home paths, personal usernames, client brands
- [ ] Skill still describes sequential + fast modes and drain gate for `/solve all`
- [ ] Relative reference loading documented so skill works outside original install path
- [ ] Grep clean; pushed to public repo

## Verification

```bash
find solve -type f | sort
rg -n -i 'teton|tw-100|tw-123|davidsolheim|/Users/|~/.grok/skills/solve|inventright|ore-?max|bnf|ipwatchdog' solve || true
```

## Drift check

* Private skill may still use TW-centric examples; public must not
* `/implement` may be external/bundled — document dependency without private paths

## Risks / blockers

* `custom-implement-instructions.md` may encode private review standards; rewrite to general high-rigor standards
* Fast mode depends on worktree isolation tooling available in the host agent — keep as optional/capability-gated language if needed

## Platform / stack

* Agent skill + git worktrees + Linear MCP (generic)

## Assumptions

* Consumers have Linear MCP + git; branch names `dev`/`main` remain the documented convention (portable, not PII)
