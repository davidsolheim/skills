# Tidy actions

Apply in the order listed in `/tidy` Phase 3. **High confidence → write now.**
Low confidence → list in the report; do not close.

Do not implement application code.

## 1. Completed (status only)

Evidence, strongest first:

1. Merged PR or commit **on `origin/main`** (or the repo’s trunk) that
   implements this identifier / AC.
2. Commit or closeout on **`origin/dev`** (pushed), not on main.
3. Linear comments already citing an `origin/dev` SHA from `/solve`.
4. Current **`origin/main` / `origin/dev` tree** clearly satisfies the AC
   (paths exist, behavior matches). Cite paths + ref.

| Evidence | Status |
|----------|--------|
| On `origin/main` (or trunk) | **Done** |
| On `origin/dev` only (pushed) | **In Review** (if the team has it; else leave open and comment shipped-on-dev) |
| Local `dev` / unpushed branch only | **Do not** change status. Optional needs-you: “ship via /prb?” |
| In Review already and on `origin/dev` only | Inspect-only if body is ready |
| In Review already and now on `origin/main` | **Done** |

Comment with ref + sha or PR url when you change status. Never Done without
main/trunk evidence.

## 2. Epic rollup

If the issue has children and **every** child is Done / Canceled / Duplicate /
Completed: comment (list child ids if short) and set the epic **Done**.
Same rule as `/solve` Phase 2E rollup. Do not implement the epic shell.

If some children are still open: do not close the epic. You may still upgrade
the epic’s short description (leaves stay the solve targets).

## 3. Duplicate / obsolete (high confidence only)

**Duplicate** when the same user-visible outcome and the same primary paths
are already tracked on another issue (open or Done). Set state **Duplicate**,
comment `Duplicate of TEAM-N`, set `relatedTo` if the API allows.

**Canceled** when the ticket targets a fully abandoned stack or a feature the
repo/newer tickets explicitly dropped, **and** a superseding issue exists (or
the product surface is gone). Comment with the superseding id or the doc
evidence.

**Low confidence** (do not close): similar-but-not-same; maybe-obsolete
without a superseder; “we might not need this.” List under the report.

## 4. Title

Retitle when the current title is vague (`Fix bug`, `Update page`, epic name
as a leaf) or wrong after research. Follow `/issue` title rules (imperative,
area prefix ok, no trailing period, ≤ ~80 chars). Keep a good title.

## 5. Relations

Set only when obvious from bodies, comments, or shared parent:

- `relatedTo` overlap that is not a duplicate
- `blockedBy` when B’s AC cannot be true until A lands
- `parent` when a leaf clearly belongs under an existing epic

Do not invent an epic. Do not clear relations you are unsure about.

## 6. Thin → `/issue` bar

Authority: `$ISSUE_SKILL_MD` (+ `issue-body-template.md` /
`execution-ready-bar.md` if present). Optional sibling:
`identify/references/upgrade.md` (same bar; Identify limits to a batch —
Tidy applies it to **every due** issue).

**Ready** (no body write) if all exist: code map of real paths, checklist AC,
repo verification commands, ≥3 drift anchors, ordered plan or file-by-file
list.

**Thin** → investigate (read-only), update the **existing** issue. Do not
create a replacement ticket. Do not change priority unless it is obviously
mislabeled vs a broken primary path (promote) or a title-only Urgent chore
(leave it; list in report).

Fail closed: no half-rewritten body.

## 7. Needs-you

Use when research cannot finish the contract:

- Product / UX decision you must not invent
- Missing credentials or third-party access
- No code pin after a reasonable search
- Status looks shipped but git evidence is ambiguous

Write **one precise question** per issue into `NEEDS_YOU`. Leave status.
Do not set Blocked. Still stamp `needs-you` so the weekly cooldown applies.

## Claimed issues

If In Progress + live foreign `claimed-by:` (< 60 min, other run) **or**
assigned to someone else: **no actions in this file**. Caller skips.
