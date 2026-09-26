---
id: "0113"
title: "Publish sanitized /issues skill to davidsolheim/skills"
status: done
priority: high
assignee:
lease_expires:
scope: "Imported from Linear TWV-113. Stay inside that description."
acceptance: "Full issues/ package on main including all four reference files"
files: []
commit:
reason:
created: "2026-07-31T14:20:05.404Z"
linear_id: "TWV-113"
linear_url: "https://linear.app/teton-web-ventures/issue/TWV-113/publish-sanitized-issues-skill-to-davidsolheimskills"
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
linear_updated: "2026-07-31T15:07:32.888Z"
linear_archived: false
notion_page_id: "3e71027c-242b-81d0-8f72-e11d5b480c20"
notion_url: "https://app.notion.com/p/3e71027c242b81d08f72e11d5b480c20"
---

## Linear import

- Identifier: TWV-113
- URL: https://linear.app/teton-web-ventures/issue/TWV-113/publish-sanitized-issues-skill-to-davidsolheimskills
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
- Created: 2026-07-31T14:20:05.404Z
- Updated: 2026-07-31T15:07:32.888Z
- Completed: 2026-07-31T15:07:32.868Z
- Canceled: no
- Archived: no
- Branch: david/twv-113-publish-sanitized-issues-skill-to-davidsolheimskills

Queue status follows Water Cooler Protocol. Todo, In Progress, In Review, Triage, and Backlog are `open` so the import does not take a ticket lease or start a review. Done, Canceled, and Blocked use those folders. `linear_status` is the Linear status at import.

## Description

## Summary

Port the private Grok `/issues` bulk-intake skill into the public [davidsolheim/skills](<https://github.com/davidsolheim/skills>) repo. Preserve multi-item decomposition, epic/`blockedBy`/`relatedTo` relations, and the shared quality bar with `/issue`, while removing client-specific Linear defaults and monorepo package path assumptions.

## User report

Publish custom Grok skills publicly with no PII and no local project references.

## Current behavior

Source skill (private):

* `issues/SKILL.md`
* `issues/references/decomposition.md`
* `issues/references/epic-body-template.md`
* `issues/references/batch-plan.md`
* `issues/references/issue-body-template.md`

**Known non-portable content (must fix before publish):**

* Explicit inventRight workspace rules: team **inventRight** (`INV`), packages `inventright-com`, `gateway-match`, `agents`
* Package-boundary language that assumes that monorepo layout
* Any client product names in examples/titles guidance

## Expected behavior

Public layout:

```text
issues/
  SKILL.md
  references/
    decomposition.md
    epic-body-template.md
    batch-plan.md
    issue-body-template.md
```

Skill must:

* Resolve Linear team/project only from **consumer repo docs** + Linear list APIs (same priority order as `/issue`)
* Use generic monorepo examples (`apps/web`, `packages/api`) instead of client package names
* Keep flags: `--draft`, `--plan-only`, `--no-epic`, `--epic`, `--max N`
* Keep atomic leaf decomposition + relation rules

## Suspected root cause / scope

Content port + replace hard-coded inventRight resolution block with generic resolution (reuse `/issue` Phase 1 language).

## Code map

| Path | Role |
| -- | -- |
| Private `issues/SKILL.md` | Bulk intake workflow; Phase 1 currently client-specific |
| Private `issues/references/*` | Decomposition + templates |
| Public `issues/` | Publish target |

## Implementation notes

1. Copy full tree into public repo.
2. Rewrite Phase 1 “resolve team/project” to remove inventRight hard-coding; point to AGENTS.md / `.linear-project` / list_teams / list_projects only.
3. Replace package path examples with neutral monorepo placeholders.
4. Align body templates with `/issue` public template quality (generic identifiers).
5. Cross-link to `/issue` and `/solve` by skill name only (not private paths).

## Acceptance criteria

- [ ] Full `issues/` package on `main` including all four reference files
- [ ] No inventRight / INV / gateway-match / inventright-com required defaults
- [ ] No personal or client PII; no absolute home paths
- [ ] CLI flags and multi-issue filing workflow preserved
- [ ] Grep clean for client brands and private monorepo paths
- [ ] Pushed to public `davidsolheim/skills`

## Verification

```bash
find issues -type f
rg -n -i 'inventright|gateway-match|\bINV\b|davidsolheim|/Users/|teton web platform|ore-?max|bnf' issues || true
```

## Drift check

* Private source still has inventRight defaults — public must not
* Templates may diverge from `/issue`; prefer shared generic wording where duplicated

## Risks / blockers

* Stripping so much that agents stop resolving Linear correctly — keep the generic auto-resolve algorithm intact

## Platform / stack

* Markdown skill package for Grok / agent skill installers

## Assumptions

* Can ship independently of `/issue` but should stay API-compatible with `/solve` leaf quality expectations
