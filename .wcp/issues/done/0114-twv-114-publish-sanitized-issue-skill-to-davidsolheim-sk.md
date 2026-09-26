---
id: "0114"
title: "Publish sanitized /issue skill to davidsolheim/skills"
status: done
priority: high
assignee:
lease_expires:
scope: "Imported from Linear TWV-114. Stay inside that description."
acceptance: "issue/SKILL.md + issue/references/issue-body-template.md exist on main"
files: []
commit:
reason:
created: "2026-07-31T14:20:05.827Z"
linear_id: "TWV-114"
linear_url: "https://linear.app/teton-web-ventures/issue/TWV-114/publish-sanitized-issue-skill-to-davidsolheimskills"
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
linear_updated: "2026-07-31T15:09:39.512Z"
linear_archived: false
notion_page_id: "3e71027c-242b-81e1-8013-db4b75e43255"
notion_url: "https://app.notion.com/p/3e71027c242b81e18013db4b75e43255"
---

## Linear import

- Identifier: TWV-114
- URL: https://linear.app/teton-web-ventures/issue/TWV-114/publish-sanitized-issue-skill-to-davidsolheimskills
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
- Created: 2026-07-31T14:20:05.827Z
- Updated: 2026-07-31T15:09:39.512Z
- Completed: 2026-07-31T15:09:39.485Z
- Canceled: no
- Archived: no
- Branch: david/twv-114-publish-sanitized-issue-skill-to-davidsolheimskills

Queue status follows Water Cooler Protocol. Todo, In Progress, In Review, Triage, and Backlog are `open` so the import does not take a ticket lease or start a review. Done, Canceled, and Blocked use those folders. `linear_status` is the Linear status at import.

## Description

## Summary

Port the private Grok `/issue` skill into the public [davidsolheim/skills](<https://github.com/davidsolheim/skills>) repo as a portable, PII-free skill package. Preserve the rapid-fire intake contract (one description → one implementation-ready Linear issue) while removing any personal, client, or local-project coupling.

## User report

Publish custom Grok skills to the public `skills` repo with no personal identifiable references and no references to local projects.

## Current behavior

Source skill lives privately under the local Grok skills tree as:

* `issue/SKILL.md`
* `issue/references/issue-body-template.md`

Known personalization / non-portable patterns to scrub or generalize:

* Example issue prefixes such as `BNF`, `TET` presented as if canonical
* Paths like `~/.grok/memory/` and other home-relative agent paths (rephrase as “agent memory / project docs” without home paths)
* Any inventRight / client monorepo examples if present in templates
* Hard-coded Linear MCP tool name assumptions that imply a single workspace layout are OK only if framed generically (`linear__save_issue` is fine; client team names are not)

## Expected behavior

Public skill directory layout (suggested):

```text
issue/
  SKILL.md
  references/
    issue-body-template.md
```

Committed on `main` of [https://github.com/davidsolheim/skills](<https://github.com/davidsolheim/skills>) with:

* Full workflow preserved (Phase 0–6, quality checklist, anti-patterns, Linear failure fallback)
* Examples use placeholder prefixes (`TEAM-123`, `ACME-42`) not real client keys
* Linear resolution remains **repo-driven** (AGENTS.md, `.linear-project`, list_teams/list_projects) with **no** required client/workspace defaults
* Zero personal names, emails, local absolute paths, private product names, or client monorepo package paths

## Suspected root cause / scope

This is a content port + sanitization task, not a behavior redesign. Copy source skill, rewrite examples and any local-only path language, verify with a grep pass for forbidden patterns.

## Code map

| Path (source) | Role |
| -- | -- |
| Private Grok skills: `issue/SKILL.md` | Main skill contract + workflow |
| Private Grok skills: `issue/references/issue-body-template.md` | Linear description template |
| Target: `issue/` under [github.com/davidsolheim/skills](<http://github.com/davidsolheim/skills>) | Public publish location |

Local clone: `~/github/skills` (or equivalent) tracking `origin/main`.

## Implementation notes

1. Copy skill tree into the public repo under `issue/`.
2. Sanitize:
   * Replace real issue-key examples (`BNF-123`, `TW-…`) with generic `TEAM-123` / `PROJ-42`
   * Replace personal home paths with portable phrasing (“agent memory files”, “workspace docs”)
   * Keep MCP tool names (`linear__save_issue`, etc.) — they are product APIs, not PII
   * Keep generic stack names only as **examples** of platform detection (Neon/ClickHouse) if they illustrate supersession; avoid framing them as this author’s required stacks
3. Optional root README section later; this issue only needs the skill package itself.
4. Do not publish private memory, session logs, or Doppler/env material.

## Acceptance criteria

- [ ] `issue/SKILL.md` + `issue/references/issue-body-template.md` exist on `main`
- [ ] Skill frontmatter `name: issue` and description suitable for public install
- [ ] `rg` / manual review: no personal names, emails, client brand names as required defaults, absolute `/Users/…` paths, or private repo names
- [ ] Examples use generic Linear identifiers only
- [ ] Operating contract still matches: one description → one Linear issue; no implement/PR under this skill
- [ ] Commit(s) pushed to public `davidsolheim/skills`

## Verification

```bash
# from public skills repo
find issue -type f
rg -n -i 'davidsolheim|/Users/|teton|inventright|ore-?max|bnf|ipwatchdog|big.?nate' issue || true
# should return no hits (or only intentional generic docs if any)
```

## Drift check

* Source skill still at private Grok skills `issue/`
* Target repo still empty or only other skill packages

## Risks / blockers

* Over-sanitizing until the skill becomes vague — keep concrete MCP steps
* Accidentally copying other private skills in the same commit

## Platform / stack

* GitHub public repo + markdown skills (no app runtime)

## Assumptions

* Team for this project is **Teton Web Ventures** (tracking only); published skill content must not hard-code that team as a skill default
* Install method for consumers can be documented later; this ticket is content publish
