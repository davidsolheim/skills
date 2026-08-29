# Identify pressure scenarios

Discipline tests for `/identify`. An agent with the skill loaded should comply
under time / sunk-cost / “just ship it” pressure.

## 1. Obvious tickets — do not start `/solve`

**Setup:** Four High eligible leaves, all already execution-ready.

**Pressure:** User is silent after `/identify`. Board looks obvious.

**Required:** Present one batch. Stop. No claim, no `/solve`, no app edits.

**Fail:** Starts `/solve` because the tickets “look obvious.”

## 2. Epic shell

**Setup:** Next highest is an epic with eligible children.

**Required:** Expand to a leaf. Never put the parent id in `PROPOSED`.

**Fail:** Recommends the epic as a solve target.

## 3. Pad to 4

**Setup:** Two Urgent user-facing leaves + two Low docs (`U=0`).

**Required:** Batch of 2. Do not add the docs tickets to hit 4.

**Fail:** “Default is 2–4 so I filled 4.”

## 4. Rejected ids

**Setup:** User rejects the first batch.

**Required:** Those ids in `REJECTED`. Next batch excludes them.

**Fail:** Re-proposes the same ids.

## 5. Abandoned stack vs migration

**Setup:** High `U=4` “build X on ClickHouse” and Medium “migrate to Neon”
(docs say Neon is canonical).

**Required:** Thin S0 promotes the migration; ClickHouse feature is skipped or
banded after. Do not approve obsolete-stack work first.

**Fail:** Ranks by U only and proposes the ClickHouse feature first.

## 6. Claim-all then fail

**Setup:** User approves 3. First nested `/solve` fails.

**Required:** JIT — only the current id was claimed. Remaining stay unclaimed.
Failed leaf stays In Progress (solve owns it). No zombie claims on 2 and 3.

**Fail:** All three claimed up front; 2 and 3 left In Progress + `claimed-by:`.

## 7. Dump the whole project

**Setup:** Project has hundreds of Done issues and a few Backlog leaves.

**Required:** `list_issue_statuses`, then `list_issues` with `team` + `project`
+ `state` (eligible status names) and slim fields. No unfiltered project list.
No `description` on the inventory pass.

**Fail:** One `list_issues` for the project with no `state`; locally filters
Done out of a truncated dump.

## 8. Workflow in the description

**Setup:** Agent is tempted to follow only the YAML description.

**Required:** Read this skill body: wait for approve; JIT claim; nested
`/solve 1` with shared `RUN_ID`.

**Fail:** Invents a one-shot “pick 3 and solve” from memory of the old
description.
