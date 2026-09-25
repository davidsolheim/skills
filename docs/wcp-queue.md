# WCP issue queue

The local queue for `/issue`, `/issues`, `/solve`, `/identify`, `/stat`, `/tidy`, `/project-review`, `/walk`, `/start`, `/prb`, and `/yeet` is `.WCP/issues/` in the current git checkout. Notion status for that same work is [`notion-issues.md`](notion-issues.md).

Do not call Linear or GitHub Issues. Do not resolve a team or a project. Do not post `claimed-by` comments.

Claim, renew, reclaim, close, cancel, and block are the player skill `water-cooler-protocol`, section Issues. This file is how those skills read and write the queue. Do not invent a second lease.

## Layout

```
.WCP/issues/open/
.WCP/issues/in-progress/
.WCP/issues/in-review/
.WCP/issues/done/
.WCP/issues/canceled/
.WCP/issues/blocked/
```

`status` in the file is the source of truth. The folder matches it. Update `status`, then move the file.

## Frontmatter

```yaml
---
id: 0123
title: Rotate refresh token on session mint
status: open | in-progress | in-review | done | canceled | blocked
priority: low | normal | high | critical
assignee:
lease_expires:
scope:
acceptance:
files: []
commit:
reason:
created: 2026-09-24T18:04:00Z
---
```

The body is the spec. `/issue` and `/issues` put the execution-ready contract in the body. `acceptance` is the short done line. `scope` is what the ticket may change. `created` is ISO-8601 UTC.

## Ids

Scan every `*.md` under `.WCP/issues/`. The next id is one greater than the highest numeric `id`, zero-padded to 4 digits. Filename: `0123-short-slug.md`.

A pin such as `0123` or `TEAM-123` is the issue whose `id` or filename contains that number. Notion uses that same id.

## Priority

| Words | priority |
| --- | --- |
| urgent, critical, broken, P0 | critical |
| high, P1 | high |
| medium, normal, default | normal |
| low, nice to have | low |

## Read the board

List the markdown files. Read frontmatter unless the body is required.

| Folder | Meaning |
| --- | --- |
| `open/` | Unclaimed. Eligible to pick. |
| `in-progress/` | Ticket lease. If `lease_expires` is past, reclaim to `open/` (player skill). |
| `in-review/` | Solver finished. Orchestrator launches one reviewer. Do not claim it as new work. |
| `blocked/` | Still wanted. Do not claim. Read `reason`. |
| `canceled/` | Will not be done. Read `reason`. |
| `done/` | Reviewer passed. `commit` is filled when the orchestrator commits the work. |

Eligible to implement: `status: open`, after expired leases are reclaimed. Skip `blocked`, `canceled`, `in-review`, and `done`. Skip `in-progress` while `lease_expires` is in the future and `assignee` is someone else.

A hard dependency is a file in `blocked/` whose `reason` names the blocker id (`blocked by 0122`). When `0122` is `done` or `canceled`, unblock: set `status: open`, clear the lease, move to `open/`, leave `reason`.

There is no epic. `in-review` is the status after the solver finishes and before the reviewer sets `done`. Do not file a parent whose only job is to hold children. Put an initiative name in the leaf body if a batch needs one.

`/solve today`: local date of `created` is today. If `created` is empty, use the file's git log date.

Area scope: title or body contains the query. There are no milestones or labels.

## File a new issue

`/issue`, `/issues`, `/project-review`, `/walk`, and `/start`:

1. Search `open/`, `in-progress/`, and `blocked/` before creating. An existing match is not filed again.
2. An older open ticket that contradicts this one: cancel it with `reason` naming the new id. Do not cancel a live `in-progress` lease.
3. Write the new file in `open/` with `status: open`, empty `assignee`, `lease_expires`, `commit`, and `reason`.
4. A hard dependency: write the blocker in `open/` first. Write the dependent in `blocked/` with `reason: blocked by <id>`.
5. Do not assign and do not set `in-progress` while filing.
6. These skills do not commit product code. Leave the new issue file in the worktree so the next queue commit includes it.
7. Upsert the Notion row for that file ([`notion-issues.md`](notion-issues.md)).

## Claim and close

`/solve` and `/identify` claim with the player skill ticket lease. One ticket per agent. Re-read after the write. If `assignee` is not you, stop.

The solver does not commit and does not stash. When acceptance is met, the solver moves the file to `in-review/` (player skill, Close). The orchestrator launches one reviewer per file in that directory. The reviewer checks security, accessibility, functionality, and aesthetics, fixes failures under a file lease, and sets `done`. The orchestrator commits only after that reviewer has exited and `wcp look` shows no live source-file lease, then writes the hash into `commit`. After each of those file moves, the orchestrator updates Notion. That update does not set Notion `done`.

On failure before review, leave the ticket `in-progress` if you still hold the lease, or `blocked` with `reason` when a human has to answer. The reviewer is the one who sets `done`.

## Ship

`/prb` and `/yeet` are the human export. They commit only when `wcp look` shows no live source-file lease. They do not stash another writer's files. Commit `.WCP/issues/` with the work. A done issue whose `commit` is in the ship stays `done`.

Immediately after `origin/dev` is pushed, update Notion with the dev SHA and the PR URL. Immediately after `origin/main` and that skill's completion gate, set Notion Status `done` ([`notion-issues.md`](notion-issues.md)).

## Stat and tidy

`/stat` reads files and does not write them. `/tidy` edits the body or frontmatter in place, then moves the file if `status` changed. Skip a ticket whose lease is still live and whose `assignee` is someone else.
