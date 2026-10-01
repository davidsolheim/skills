---
name: water-cooler-protocol
description: >
  Occupancy leases for many coding agents on one shared local checkout,
  and the committed work queue in .wcp/issues/.
  Load when this repo's AGENTS.md mentions WCP, when WCP_AGENT is set, when
  coordinating parallel agents on local dev, or when the user runs /wcp.
  Write tests and new files directly. Claim a pre-existing file only after
  a test names it. Never rewind sibling edits. Never push origin/dev as an agent.
---

# Water Cooler Protocol (player)

You are one writer among others on this checkout. Changed files are other workers, not corruption.

Spec: if this repo contains `PROTOCOL.md` for WCP, that file is the protocol. Commands below are the verbs. Skill wiring for init, ids, solve waves, ticket occupancy, and the ship push: [`../docs/wcp.md`](../docs/wcp.md).

## Identity

Name yourself. Do not wait for a human to assign the id. Prefer a short lowercase name from the work (`session-refresh`, or the issue id `issue-123`).

```bash
wcp name session-refresh --json
export WCP_AGENT=<agent>
export WCP_NAME_TOKEN=<token>
```

If `WCP_AGENT` is already set, `wcp name` that id. `name_taken` means pick a different id and export the new pair. Once `WCP_NAME_TOKEN` is set, that is your name for the run. Do not take a second id. Do not call `wcp set-arch` or `wcp stop`.

## Start

Read `.wcp/issues/open/`, `.wcp/issues/in-progress/`, and `.wcp/issues/in-review/`. If the checkout has a `.WCP/` directory and no `.wcp/` directory, the queue is `.WCP/issues/` even when that issues folder does not exist yet. Create it there. Do not create `.wcp/` while `.WCP/` is the live directory. If both directories exist, do not start `wcp` and do not pick one. Stop any daemon. Do not rename one folder onto the other. Do not rename it during a run. Reclaim expired `in-progress` tickets (Issues, below). Do not reclaim `in-review`. Do not copy tickets, specs, or diffs onto `.wcp/RUN.md`. `arch` on the board stays the session aim, not a copy of every ticket.

If those directories are missing and this run needs a queue, create `open/`, `in-progress/`, `in-review/`, `done/`, `canceled/`, and `blocked/`. File one stamped issue for this session. File another file when the session contains a separate change. Do not invent a backlog.

## Turn

```
wcp name <id> --json
# export WCP_AGENT and WCP_NAME_TOKEN
wcp look --json
# 1. Write the test. Do not acquire it.
#    // WCP <id>: <existing-path> <what it proves> (<arch>)
# 2. New file (not in the tree when the run started): write it. Stop. No acquire.
# 3. File that already existed:
wcp acquire --path <existing-file> --test <test-file> --doing "<burst>" --scope "<symbol>" --json
# research in this file, including between hunks
wcp reup --json
wcp write-ok --path <existing-file> --json
# re-read that file from disk, edit, flush
# wcp reup while you stay in the file
wcp release --json
# run YOUR tests only
```

A claim is only for a file that was already in the tree. Write a test file directly. Write a new file directly. You are the only writer on a path you are creating.

The seat means you are in that file, including research between hunks. The next hunk may still be in an unflushed buffer. Heartbeat with `wcp reup` while you stay. It refreshes `expires_at` by the takeover window (300 seconds). It does not require a write. Release when you leave the file. Holding the seat through tests, or through work on a different file, is a protocol break. One live source path per agent. One agent per path. If you must enter that file again, acquire again.

Do not edit `.wcp/RUN.md`. Call `wcp`. `wcp look` is the observer view. It stays cheap and does not hash the tree.

## Tests

Before any claim, write or append the test. The comment names the file you will claim:

```
// WCP <id>: <path> <what it proves> (<arch>)
```

(`# WCP` / `-- WCP` in other languages.) `acquire` fails with `no_test` until that line is on disk in a test file written during this run. Prefer a per-slice test file. Never delete someone else's tests. Never hollow a test to make your slice green. Foreign red is not your write. Full-suite green is the human's gate, not yours.

## Conflict (mandatory order)

On `conflict`, read `existing.doing` / `existing.scope`:

1. Same symbol or same intent → retarget. Do not overtake a live seat.
2. The seat is takeable — no write and no heartbeat for 5 minutes, or the recorded pid is dead → `wcp overtake --path <file> --test <test-file>` only to **finish** that work. Your test must name the path. Inherit `doing` and `scope`. Set `from_agent`. Do not revert hunks. Do not swap designs.
3. Else pick another path from `arch`. Do not busy-loop `look`. A research gap inside the file is not a reason to overtake.

`test_file` means write the test and do not claim it. `new_file` means write the file and do not claim it. `no_test` means the test naming this path is not on disk yet.

If the work is wrong, do not overtake. Leave it.

## Drift

If `write-ok` returns `drift` on a path inside your scope: stop. `wcp look`. Retarget. Never `git reset --hard`, `git checkout --`, or restore to "go first."

## Git

Never commit the board or sqlite. The orchestrator is the only one who runs `git commit`. A worker never commits and never stashes. Stashing on a shared checkout hides another writer's uncommitted files. The orchestrator commits only when `wcp look` shows no live source-file lease. If a lease is live, wait. The work commit is the product paths. The next commit is the `.wcp/issues/` update.

```
.wcp/RUN.md
.wcp/run.sqlite
.wcp/*.sqlite-wal
.wcp/*.sqlite-shm
```

`wcp init` writes those lines and removes a blanket `.wcp/` or `.WCP/` ignore. Hooks reject any staged `.wcp/` or `.WCP/` path outside `issues/` (socket, lock, pid, log, mode, barrels). A checkout that has `.WCP/` and no `.wcp/` is still the queue. Use that folder. Do not rename it during a run. `wcp doctor` prints `git mv .WCP .wcp-tmp && git mv .wcp-tmp .wcp` when the folder is tracked, and `mv .WCP .wcp-tmp && mv .wcp-tmp .wcp` when it is runtime only. Do not store source in `.wcp/`. Do not push. Do not rewind. `WCP_AGENT` is set: hooks will refuse push.

Do not `wcp acquire` an issue file. A ticket lease is the frontmatter on that file. It is not a source-file lock.

## Barrels

If the path matches `.wcp/barrels` (lockfiles, generated clients, root schema): extra-short burst. No thinking on the lease.

## Board

`wcp look` before every write. Edit occupancy only through `wcp`. Do not rewrite `.wcp/RUN.md` by hand (the daemon overwrites it). `RUN.md` is occupancy and `arch` only. Do not put specs, ticket status, or diffs on it. `look` does not reap and does not hash the tree. A human may poll it every 5 seconds on an active checkout. Five minutes is how long a quiet seat is protected, not how long the seat stays invisible.

## End

When the work matches the ticket's `acceptance`, release every source-file lease, move the ticket to `in-review/`, and stop. The orchestrator launches one reviewer for that file. The reviewer sets `done`. A worker does not commit. An idle ticket lease is a protocol violation. A source seat held through tests or across files is the same kind of violation.

## Issues

Committed queue for a solo builder with many agents on one checkout. The issue file is the record. Write it before the work and before any Notion call. Do not call Linear or GitHub Issues. Assign, claim, and close work by editing files under `.wcp/issues/`. Do not call Notion during a source-file lease. After the file is written, the orchestrator or the filing or ship skill may copy it to Notion (`../docs/notion-issues.md`). If that copy fails, the file stands. Do not stop the work. Notion `done` is the ship to `origin/main`, not this close.

Every session task files at least one issue. One change that can be claimed, reviewed, and shipped on its own is one file. Two such changes are two files. They share `session`. There is no parent issue and no session directory.

### Layout

```
.wcp/issues/
  open/20261001T1202Z-0123-rotate-refresh-token.md
  in-progress/20261001T1202Z-0123-rotate-refresh-token.md
  in-review/20261001T1202Z-0123-rotate-refresh-token.md
  blocked/20261001T1202Z-0123-rotate-refresh-token.md
  done/2026/10/01/20261001T1202Z-0123-rotate-refresh-token.md
  canceled/2026/10/01/20261001T1202Z-0123-rotate-refresh-token.md
```

`open/`, `in-progress/`, `in-review/`, and `blocked/` stay flat. A new `done/` or `canceled/` file goes under `YYYY/MM/DD` from `created`. Walk every `*.md` under the status directory, including a day tree and any older flat file.

The filename is `YYYYMMDDThhmmZ-<id>-<slug>.md`. The example above is `20261001T1202Z-0123-rotate-refresh-token.md` for `created: 2026-10-01T12:02:00Z`. A status move keeps that filename. `status` inside the file is the source of truth. The folder is a projection. Update `status`, then `git mv` the file once it is tracked.

### Frontmatter

```yaml
---
id: 0123
title: Rotate refresh token on session mint
status: open | in-progress | in-review | done | canceled | blocked
priority: low | normal | high | critical
assignee:          # agent id, empty if unclaimed
lease_expires:     # ISO-8601 UTC, empty if unclaimed
scope:             # what this ticket is allowed to change
acceptance:        # short done definition
files: []          # paths touched while solving
commit:            # work-commit hash, empty until the orchestrator writes it
pr:                # pull request URL, empty until the ship knows it
reason:            # why it was canceled or blocked; empty unless status is canceled or blocked
created: 2026-10-01T12:02:00Z
session: 2026-10-01T12:00:00Z
notion_page_id:    # filled when a Notion copy succeeds; empty until then
notion_url:
---
```

The body under the frontmatter is the spec. Keep it short enough to work from.

### Notion resync

Copy the file to Notion only after the file write, and only outside a source-file lease. A failed copy does not change the file. A later resync walks `open/`, `in-progress/`, `in-review/`, and `blocked/` whole, and walks `done/` and `canceled/` by day. Upsert by issue id. Running it twice is safe. There is no sync cursor and no daemon. Fill `notion_page_id` and `notion_url` when a copy succeeds. Write `pr` on the file when the ship knows the pull request URL.

### Ticket lease

Claim an open ticket, or an expired `in-progress` ticket:

1. Read the file. If `status` is `in-progress` and `lease_expires` is still in the future, it is held. Pick another.
2. Set `assignee` to your agent id, `status: in-progress`, and `lease_expires` to now + 10 minutes, ISO-8601 UTC (shape `2026-09-24T18:04:00Z`).
3. Write the file, then move it to `in-progress/` keeping the stamped filename.
4. Re-read the file at its new path. If `assignee` is not you, stop. Do not write your id back in.

Renew while you hold it and you are still working: before `lease_expires`, set it to now + 10 minutes again. That edit does not take a source-file lease.

Reclaim when `lease_expires` is past and the assignee is not renewing. The orchestrator does this at the start of a run. Clear `assignee` and `lease_expires`, set `status: open`, move the file to `open/` keeping the stamped filename. Then re-read. If `assignee` is set and `lease_expires` is in the future, that claim stands. Do not clear it again.

Reclaim and claim are idempotent. Only one agent is `assignee`. If two writes race, the last writer re-reads and leaves a single assignee.

### While solving

The orchestrator, or you if no one is assigned, picks one file in `open/`. Skip `blocked/`, `canceled/`, and `in-review/`. One ticket per agent unless the user says otherwise. Do not claim a directory. Ticket claim is not a file claim.

Use the body as the spec. Implement with the file seats above. Tests first, then a source seat. Heartbeat with `wcp reup` while you are in the file, including research between hunks. Release before tests or before work on a different file. The takeover window is 300 seconds.

Append each touched path to `files` when it lands. Do not put the diff on `RUN.md`.

Two agents do not hold the same ticket. They do not hold the same source file. Different files on one ticket only when `scope` allows it, and each agent still has one live source file.

### Close

When the work matches `acceptance`, the solver records the paths in `files`, releases every source-file lease, and leaves the tree dirty. The solver sets `status: in-review`, clears `lease_expires`, leaves `assignee` as itself, leaves `commit` and `pr` empty, and moves the file to `in-review/` keeping the stamped filename. The solver does not commit, does not stash, and does not set `done`.

The orchestrator watches `.wcp/issues/in-review/`. For each file there, it launches one reviewer. The reviewer reads that issue and the paths in `files`, and checks security, accessibility, functionality, and aesthetics against `acceptance`.

If the check fails, the reviewer fixes the code. A pre-existing file uses the file lease: look, acquire, write-ok, edit, release. The reviewer does not commit and does not stash.

When the check passes, the reviewer sets `status: done`, clears `assignee` and `lease_expires`, and moves the file to `done/YYYY/MM/DD/` from `created`, keeping the stamped filename. `commit` stays empty until a work commit exists. `pr` stays empty until the ship knows the pull request URL.

The orchestrator does not commit while a reviewer is still running. After that reviewer has exited, and `wcp look` shows no live source-file lease:

1. Commit the work. `files` already lists the paths. `commit` cannot name a hash that does not exist yet.
2. Write that hash into `commit` on the done file. When the ship knows the pull request URL, write it into `pr` as well.
3. Commit that issue-file update. `wcp look` is still empty.

The reviewer does not commit. A solver does not commit. The orchestrator is the only one who commits.

### Cancel

Cancel when the work will not be done. The holder may cancel a ticket they hold. The orchestrator may cancel an open ticket, or an `in-progress` ticket whose lease is expired. Do not cancel a ticket another agent holds under a live lease.

1. Write `reason`. A canceled ticket with an empty `reason` is not canceled.
2. Set `status: canceled`. Clear `assignee` and `lease_expires`. Leave `commit` empty unless a work hash already exists. Keep `files` if paths already landed.
3. Move the file to `canceled/YYYY/MM/DD/` from `created`, keeping the stamped filename. Do not delete it. Do not set `done`.
4. Re-read. One file, one `reason`. If another writer canceled or reclaimed it, leave their newer `assignee` or `reason` in place.
5. The orchestrator commits the issue file, and only when `wcp look` shows no live source-file lease. Until then the file update still stands.

Search `.wcp/issues/canceled/` and `status: canceled` before filing the same work again. Read `reason`. Do not open a second ticket for it unless the user says to revive it. Reviving sets `status: open`, clears the lease, and moves the file to `open/`. Leave `reason` so the earlier cancel stays searchable.

### Block

Block when the work is still wanted and an agent must not claim it yet. The holder may block a ticket they hold. The orchestrator may block an open ticket, or an `in-progress` ticket whose lease is expired. Do not block a ticket another agent holds under a live lease.

1. Write `reason`. A blocked ticket with an empty `reason` is not blocked.
2. Set `status: blocked`. Clear `assignee` and `lease_expires`. Leave `commit` empty unless a work hash already exists. Keep `files` if paths already landed.
3. Move the file to `blocked/` keeping the stamped filename. Do not delete it. Do not set `done`. Do not set `canceled`.
4. Re-read. One file, one `reason`. If another writer blocked, canceled, or reclaimed it, leave their newer `assignee` or `reason` in place.
5. The orchestrator commits the issue file, and only when `wcp look` shows no live source-file lease. Until then the file update still stands.

Do not claim a file in `.wcp/issues/blocked/`. Search that folder and `status: blocked` before filing the same work again. Read `reason`. Unblocking sets `status: open`, clears the lease, and moves the file to `open/`. Leave `reason` so the earlier block stays searchable.

### Orchestrator

Assign work from `.wcp/issues/open/` only. Skip `blocked/`, `canceled/`, and `in-review/`. Watch `.wcp/issues/in-review/` and launch one reviewer per file, as in Close. No Linear or GitHub Issues calls. After the file write, copy it to Notion (`../docs/notion-issues.md`) outside a source-file lease. If Notion fails, the file stands. Do not set Notion `done` here. One ticket per agent unless the user says otherwise. Do not claim a directory. Two agents may share a ticket's files only under that ticket's `scope`, and WCP file rules still hold.

### Two clocks

Do not collapse them.

- File takeover is 5 minutes of no write and no heartbeat, or a dead pid (default TTL 300 seconds). `overtake` inherits `doing` and `scope`. That clock is not the ticket lease.
- Ticket reclaim is 10 minutes on `lease_expires`. Ticket leases do not acquire the issue file.
