# wcp issue queue

The local queue for `/issue`, `/issues`, `/solve`, `/identify`, `/stat`, `/tidy`, `/project-review`, `/walk`, `/start`, `/prb`, and `/yeet` is `.wcp/issues/` in the current git checkout. The player skill is `water-cooler-protocol`. There is no daemon.

wcp issues are always the record. `.wcp/tracker.md` names an optional external tracker (`notion`, `linear`, `jira`, another tool, or `none`). Mirror updates only when `tracker` is not `none`. Notion's procedure is [`notion-issues.md`](notion-issues.md). A missing file or `none` means do not call an external tracker.

Every path these skills write is under `.wcp/`.

Do not call an external tracker unless `.wcp/tracker.md` names it.

## Layout

```
.wcp/issues/open/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/in-progress/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/blocked/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/done/2026/10/01/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/deployed-dev/2026/10/01/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/deployed-main/2026/10/01/20261001T1202Z-0123-rotate-refresh-token.md
.wcp/issues/canceled/2026/10/01/20261001T1202Z-0123-rotate-refresh-token.md
```

`open/`, `in-progress/`, and `blocked/` stay flat. `done/`, `deployed-dev/`, `deployed-main/`, and `canceled/` go under `YYYY/MM/DD` from `created`. Walk every `*.md`, including a day tree and any older flat file.

Set `status`, then move the file. Keep the stamped filename. An issue left in `in-review/` is finished writing. Move it to `done/`.

## Frontmatter

```yaml
---
id: 0123
title: Rotate refresh token on session mint
status: open
assignee:
files: []
acceptance:
reason:
dev:
main:
created: 2026-10-01T12:02:00Z
session: 2026-10-01T12:00:00Z
---
```

The body is the spec. `acceptance` is the short done line. `files` is the paths touched while the issue is in `in-progress/`. `dev` is the sha on `origin/dev`. `main` is the sha on `origin/main`. Both stay empty until those commits exist. `created` is when the file was filed. `session` is when the sitting opened.

## Ids

Scan every `*.md` under `.wcp/issues/`. The next id is one greater than the highest numeric `id`, zero-padded to 4 digits. Filename: `YYYYMMDDThhmmZ-0123-short-slug.md`, from `created`.

A pin such as `0123` is the issue whose `id` or filename contains that number.

## Read the board

| Folder | Meaning |
| --- | --- |
| `open/` | Not started. Eligible to pick. Does not block a commit. |
| `in-progress/` | An agent is editing this on this machine. Read `files` and work around those paths. |
| `done/` | Writing is finished. Not pushed. |
| `deployed-dev/` | The commit is on `origin/dev`. |
| `deployed-main/` | The commit is on `origin/main`. |
| `blocked/` | Still wanted. Do not take it. Read `reason`. |
| `canceled/` | Will not be done. Read `reason`. |

Take work from `open/` only. Skip `blocked/`, `canceled/`, `done/`, `deployed-dev/`, and `deployed-main/`. If an `in-progress/` issue's agent is gone, move it back to `open/` and clear `assignee`.

A hard dependency is a file in `blocked/` whose `reason` names the blocker id (`blocked by 0122`). When `0122` is `deployed-dev`, `deployed-main`, or `canceled`, move the blocked file to `open/` and leave `reason`.

There is no parent issue. One change that can ship on its own is one file.

## File a new issue

1. Search `open/`, `in-progress/`, and `blocked/` before creating. An existing match is not filed again. Search `canceled/` and read `reason` before filing the same work again.
2. Write the new file in flat `open/` with `status: open` and empty `assignee`, `files`, `reason`, `dev`, and `main`. Set `created` to now UTC and `session` to the session open time.
3. A hard dependency: write the blocker in `open/` first. Write the dependent in `blocked/` with `reason: blocked by <id>`.
4. Do not move it to `in-progress/` while filing.
5. After the file is written, mirror it to the tracker named in `.wcp/tracker.md`. If that file is missing or `tracker` is `none`, skip. A failed mirror leaves the file as the record.

## Solve and ship

Move the issue you take to `in-progress/`. Add paths to `files` as you write them. When `acceptance` is met, move it to `done/`.

If anything else is in `in-progress/`, or you still have a task, stop. Do not commit. Do not push.

If `in-progress/` is empty and you have no further task, commit the tree, push `dev` to `origin/dev`, write that sha into `dev`, and move those `done/` issues to `deployed-dev/YYYY/MM/DD/`.

A later run moves an issue to `deployed-main/` only after that commit is on `origin/main`, and writes the sha into `main`.

`/stat` reads files and does not write them. `/tidy` edits a file in place, then moves it if `status` changed. Do not tidy an `in-progress/` issue whose `assignee` is someone else.
