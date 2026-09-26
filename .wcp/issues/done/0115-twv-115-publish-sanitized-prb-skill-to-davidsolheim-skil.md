---
id: "0115"
title: "Publish sanitized /prb skill to davidsolheim/skills"
status: done
priority: high
assignee:
lease_expires:
scope: "Imported from Linear TWV-115. Stay inside that description."
acceptance: "prb/SKILL.md + prb/references/db-migrations.md on public main"
files: []
commit:
reason:
created: "2026-07-31T14:20:08.022Z"
linear_id: "TWV-115"
linear_url: "https://linear.app/teton-web-ventures/issue/TWV-115/publish-sanitized-prb-skill-to-davidsolheimskills"
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
linear_updated: "2026-07-31T15:12:33.818Z"
linear_archived: false
notion_page_id: "3e71027c-242b-8152-a66b-c4cb2a40e0fc"
notion_url: "https://app.notion.com/p/3e71027c242b8152a66bc4cb2a40e0fc"
---

## Linear import

- Identifier: TWV-115
- URL: https://linear.app/teton-web-ventures/issue/TWV-115/publish-sanitized-prb-skill-to-davidsolheimskills
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
- Created: 2026-07-31T14:20:08.022Z
- Updated: 2026-07-31T15:12:33.818Z
- Completed: 2026-07-31T15:12:33.798Z
- Canceled: no
- Archived: no
- Branch: david/twv-115-publish-sanitized-prb-skill-to-davidsolheimskills

Queue status follows Water Cooler Protocol. Todo, In Progress, In Review, Triage, and Backlog are `open` so the import does not take a ticket lease or start a review. Done, Canceled, and Blocked use those folders. `linear_status` is the Linear status at import.

## Description

## Summary

Port the private Grok `/prb` ship skill (push `dev` → PR into `main` → babysit CI → optional prod migrate → merge) into the public [davidsolheim/skills](<https://github.com/davidsolheim/skills>) repo without personal identifiers, client project names, or local-only migration procedures presented as required defaults.

## User report

Publish custom Grok skills publicly with no PII and no local project references.

## Current behavior

Source skill (private):

* `prb/SKILL.md`
* `prb/references/db-migrations.md`

**Known non-portable patterns:**

* Linear example ids `TW-123`
* Migration discovery may reference specific private stacks or Doppler/org habits — keep as **discovery algorithm** (“find this repo’s documented production migrate path”), never hard-code a client’s procedure
* Any absolute paths or private product names in migration reference

## Expected behavior

Public layout:

```text
prb/
  SKILL.md
  references/
    db-migrations.md
```

Preserve operating contract:

1. Refresh from `origin/main` before push
2. Merge `origin/main` into local `dev`
3. Push `origin/dev`
4. Open/reuse PR `main` ← `dev`
5. Babysit CI/bots (default 5 min interval, 15 min window)
6. Production migrations only when ship includes them, via **repo-documented** procedure
7. Auto-merge only when quiet window + green CI + migrations done (unless `--no-merge`)

Sanitize:

* Generic Linear issue id examples
* Migration doc must be stack-agnostic discovery (package.json scripts, README, AGENTS.md, CI jobs) with explicit ban on inventing `db:push` to prod

## Suspected root cause / scope

Small package (2 files). Main risk is `db-migrations.md` leaking private deploy/migrate runbooks.

## Code map

| Path | Role |
| -- | -- |
| Private `prb/SKILL.md` | Ship/babysit/merge workflow |
| Private `prb/references/db-migrations.md` | Migration gate discovery |
| Public `prb/` | Publish target |

## Implementation notes

1. Copy both files.
2. Replace `TW-123` examples with `TEAM-123`.
3. Rewrite migration reference to pure discovery checklist; remove any client-specific Neon/Railway/Vercel commands if they encode one org’s standard without labeling as example-only.
4. Keep branch naming convention `dev`/`main` (generic workflow, not PII).
5. Mention `/solve` as upstream local-dev producer by skill name only.

## Acceptance criteria

- [ ] `prb/SKILL.md` + `prb/references/db-migrations.md` on public `main`
- [ ] No TW-/client/personal paths; no private prod runbooks as hard requirements
- [ ] Flags preserved: `--no-merge`, `--skip-migrations`, watch/interval overrides
- [ ] Hard rule remains: never push `dev` without merging latest `main` first
- [ ] Grep clean; pushed to public repo

## Verification

```bash
find prb -type f
rg -n -i 'tw-123|davidsolheim|/Users/|teton|inventright|ore-?max|doppler.*prod|bnf' prb || true
```

## Drift check

* Private migration doc may still list preferred hoster scripts — public must stay discovery-first

## Risks / blockers

* Making migrations too vague; keep a concrete discovery order and safety bans

## Platform / stack

* GitHub CLI / git + optional CI babysitting; migrations discovered per repo

## Assumptions

* `gh` available for PR create/merge in consumer environments; skill should already state failure modes if not
