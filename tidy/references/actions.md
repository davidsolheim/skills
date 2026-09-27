# Tidy actions

Apply in the order listed in `/tidy` Phase 3. **High confidence → write now.**
Low confidence → list in the report; do not close.

Do not implement application code. Do not call Linear. Edit the issue file, then update Notion ([`../../docs/notion-issues.md`](../../docs/notion-issues.md)).

## 1. Completed (status only)

Evidence, strongest first:

1. Merged commit **on `origin/main`** (or the repo’s trunk) that implements this id / acceptance.
2. Commit on **`origin/dev`** (pushed), not on main.
3. The issue file’s `commit` already names that sha.
4. Current **`origin/main` / `origin/dev` tree** clearly satisfies acceptance. Cite paths + ref.

| Evidence | File `status` | Notion |
|----------|---------------|--------|
| On `origin/main` (or trunk) | `done` | `done` |
| On `origin/dev` only (pushed) | `in-review` | `in-review` |
| Local `dev` / unpushed only | **Do not** change status | unchanged |
| Already `in-review` and on `origin/dev` only | Inspect-only if the body is ready | unchanged if already `in-review` |
| Already `in-review` and now on `origin/main` | `done` | `done` |

Move the file to the folder that matches `status`. Never set `done` without main/trunk evidence. Notion `done` follows that same evidence.

## 2. Parent shells

There is no epic file ([`../../docs/wcp-queue.md`](../../docs/wcp-queue.md)). If a file has no acceptance of its own and only names other ids, and every named id is `done` or `canceled`, set this file `canceled` with `reason` listing those ids. Do not invent a parent.

## 3. Duplicate / obsolete (high confidence only)

**Duplicate** when the same user-visible outcome and the same primary paths are already tracked on another file (open or done). Set `status: canceled`, `reason: duplicate of 0123`, and move the file to `canceled/`. Set Notion `canceled`.

**Canceled** when the ticket targets a fully abandoned stack or a feature the repo or a newer ticket explicitly dropped, **and** a superseding id exists (or the product surface is gone). `reason` names the superseding id or the doc.

**Low confidence** (do not close): similar-but-not-same; maybe-obsolete without a superseder. List it in the report.

Do not cancel a file whose lease is live and whose `assignee` is someone else.

## 4. Title

Retitle when the current title is vague (`Fix bug`, `Update page`) or wrong after research. Follow `/issue` title rules (imperative, area prefix ok, no trailing period, ≤ ~80 chars). Keep a good title. Update the Notion **Name** when the title changes.

## 5. Relations

Set only when obvious from bodies or shared scope:

- Name a related id in the body when the overlap is not a duplicate
- `blocked/` + `reason: blocked by 0122` when this acceptance cannot be true until `0122` lands

Do not invent a parent. Do not clear a `reason` you are unsure about.

## 6. Thin → `/issue` bar

Authority: `$ISSUE_SKILL_MD` (+ `issue-body-template.md` / `execution-ready-bar.md` if present). Optional sibling: `identify/references/upgrade.md` (same bar; Identify limits to a batch — Tidy applies it to **every due** issue).

**Ready** (no body write) if all exist: code map of real paths, checklist acceptance, repo verification commands, ≥3 drift anchors, ordered plan or file-by-file list, and `## Occupancy (WCP)` with a primary write path (or explicit N/A when the leaf writes no application files). Missing occupancy is thin.

**Thin** → investigate (read-only), update the **existing** file. Do not create a replacement. Do not change `priority` unless it is obviously mislabeled vs a broken primary path (promote) or a title-only critical chore (leave it; list in the report). When you **do** rewrite, fill Occupancy (WCP) from the `/issue` template ([`../../docs/wcp.md`](../../docs/wcp.md)).

Fail closed: no half-rewritten body. After a body or title write, upsert Notion.

## 7. Needs-you

Use when research cannot finish the contract:

- Product / UX decision you must not invent
- Missing credentials or third-party access
- No code pin after a reasonable search
- Status looks shipped but git evidence is ambiguous

Write **one precise question** per issue into `NEEDS_YOU`. Leave `status`. Do not set `blocked` for a question. Still stamp `needs-you` so the weekly cooldown applies.

## Claimed issues

If `in-progress/` and `lease_expires` is in the future and `assignee` is someone else: **no actions in this file**. The caller skips.
