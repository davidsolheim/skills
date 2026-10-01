# WCP issue queue

The local queue for `/issue`, `/issues`, `/solve`, `/identify`, `/stat`, `/tidy`, `/project-review`, `/walk`, `/start`, `/prb`, and `/yeet` is `.wcp/issues/` in the current git checkout. Notion status for that same work is [`notion-issues.md`](notion-issues.md).

Every path these skills write as `.wcp/` means that folder. If the checkout has `.WCP/` and no `.wcp/`, read and write `.WCP/` instead, including when `issues/` does not exist yet. Do not create `.wcp/` while `.WCP/` is the live directory. If both directories exist, stop. Do not rename one folder onto the other, and do not rename during a run.

Do not call Linear or GitHub Issues. Do not resolve a team or a project. Do not post `claimed-by` comments.

Claim, renew, reclaim, close, cancel, and block are the player skill `water-cooler-protocol`, section Issues. This file is how those skills read and write the queue. Do not invent a second lease.

## Layout

```
.wcp/issues/open/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/in-progress/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/in-review/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/blocked/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/done/2026/10/01/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/canceled/2026/10/01/20261001T1202Z-0123-rotate-refresh-token.md
```

On a checkout that still has only `.WCP/`, those same paths are under `.WCP/issues/`.

`open/`, `in-progress/`, `in-review/`, and `blocked/` stay flat. A new `done/` or `canceled/` file goes under `YYYY/MM/DD` from `created`. Walk every `*.md` under the status directory, including a day tree and any older flat file.

`status` in the file is the source of truth. The folder matches it. Update `status`, then move the file. Keep the stamped filename.

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
pr:
reason:
created: 2026-10-01T12:02:00Z
session: 2026-10-01T12:00:00Z
notion_page_id:
notion_url:
---
```

The body is the spec. `/issue` and `/issues` put the execution-ready contract in the body. `acceptance` is the short done line. `scope` is what the ticket may change. `created` is the UTC time this file was filed. `session` is the UTC time the session opened, shared by the files from that sitting. `pr` is the pull request URL, empty until the ship knows it.

## Ids

Scan every `*.md` under `.wcp/issues/`. The next id is one greater than the highest numeric `id`, zero-padded to 4 digits. Filename: `YYYYMMDDThhmmZ-0123-short-slug.md`, from `created`. The example for `2026-10-01T12:02:00Z` is `20261001T1202Z-0123-short-slug.md`.

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
3. Write the new file in flat `open/` with `status: open`, empty `assignee`, `lease_expires`, `commit`, `pr`, and `reason`. Set `created` to now UTC and `session` to the session open time. Use the stamped filename.
4. A hard dependency: write the blocker in `open/` first. Write the dependent in flat `blocked/` with `reason: blocked by <id>`.
5. Do not assign and do not set `in-progress` while filing.
6. These skills do not commit product code. Leave the new issue file in the worktree so the next queue commit includes it.
7. After the file is written, upsert the Notion row ([`notion-issues.md`](notion-issues.md)). If Notion fails, the file stands. Do not stop filing.

## Claim and close

`/solve` and `/identify` claim with the player skill ticket lease. One ticket per agent. Re-read after the write. If `assignee` is not you, stop.

The solver does not commit and does not stash. When acceptance is met, the solver moves the file to flat `in-review/` (player skill, Close), keeping the stamped filename. The orchestrator launches one reviewer per file in that directory. The reviewer checks security, accessibility, functionality, and aesthetics, fixes failures under a file lease, and sets `done`, moving the file to `done/YYYY/MM/DD/` from `created`. The orchestrator commits only after that reviewer has exited and `wcp look` shows no live source-file lease, then writes the hash into `commit`. After each of those file moves, the orchestrator updates Notion. If Notion fails, the file stands. That update does not set Notion `done`.

On failure before review, leave the ticket `in-progress` if you still hold the lease, or `blocked` with `reason` when a human has to answer. The reviewer is the one who sets `done`.

## Ship

`/prb` and `/yeet` are the human export. They commit only when `wcp look` shows no live source-file lease. They do not stash another writer's files. Commit `.wcp/issues/` with the work. A done issue whose `commit` is in the ship stays `done`.

When the ship knows the pull request URL, write it into `pr` on the issue file. Immediately after `origin/dev` is pushed, update Notion with the dev SHA and the PR URL. Immediately after `origin/main` and that skill's completion gate, set Notion Status `done` ([`notion-issues.md`](notion-issues.md)). If Notion fails, `pr` and `commit` on the file are the record. A later resync copies them.

## Stat and tidy

`/stat` reads files and does not write them. `/tidy` edits the body or frontmatter in place, then moves the file if `status` changed. Skip a ticket whose lease is still live and whose `assignee` is someone else.
