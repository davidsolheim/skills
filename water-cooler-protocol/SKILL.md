---
name: water-cooler-protocol
description: >
  Occupancy leases for many coding agents on one shared local checkout,
  and the committed work queue in .WCP/issues/.
  Load when this repo's AGENTS.md mentions WCP, when WCP_AGENT is set, when
  coordinating parallel agents on local dev, or when the user runs /wcp.
  Write tests and new files directly. Claim a pre-existing file only after
  a test names it. Never rewind sibling edits. Never push origin/dev as an agent.
---

# Water Cooler Protocol (player)

You are one writer among others on this checkout. Changed files are other workers, not corruption.

Spec: if this repo contains `PROTOCOL.md` for WCP, that file is the protocol. Commands below are the verbs.

## Identity

Name yourself. Do not wait for a human to assign the id. Prefer a short lowercase name from the work (`session-refresh`, or the issue id `issue-123`).

```bash
wcp name session-refresh --json
export WCP_AGENT=<agent>
export WCP_NAME_TOKEN=<token>
```

If `WCP_AGENT` is already set, `wcp name` that id. `name_taken` means pick another id and export the new pair. Once `WCP_NAME_TOKEN` is set, that is your name for the run. Do not take a second id. Do not call `wcp set-arch` or `wcp stop`.

## Start

Read `.WCP/issues/open/` and `.WCP/issues/in-progress/`. Reclaim expired tickets (Issues, below). Do not copy tickets, specs, or diffs onto `.WCP/RUN.md`. `arch` on the board stays the session aim, not a copy of every ticket.

If those directories are missing and this run needs a queue, create `open/`, `in-progress/`, `done/`, `canceled/`, and `blocked/`, and one issue whose body is the current `arch`. Do not invent a backlog.

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
wcp write-ok --path <existing-file> --json
# re-read that file from disk, edit, flush
wcp release --json
# run YOUR tests only
```

A claim is only for a file that was already in the tree. Write a test file directly. Write a new file directly. You are the only writer on a path you are creating.

Lease is a burst on that existing file. Release the instant bytes are on disk — before tests, thinking, or waiting. Holding a lease through the test runner is a protocol break. If you must write that existing file again, acquire again.

`wcp reup` only while still flushing this burst (TTL about to expire mid-write).

## Tests

Before any claim, write or append the test. The comment names the file you will claim:

```
// WCP <id>: <path> <what it proves> (<arch>)
```

(`# WCP` / `-- WCP` in other languages.) `acquire` fails with `no_test` until that line is on disk in a test file written during this run. Prefer a per-slice test file. Never delete someone else's tests. Never hollow a test to make your slice green. Foreign red is not your write. Full-suite green is the human's gate, not yours.

## Conflict (mandatory order)

On `conflict`, read `existing.doing` / `existing.scope`:

1. Same symbol or same intent → retarget. Do not overtake a live burst.
2. `expired` or dead pid → `wcp overtake --path <file> --test <test-file>` only to **finish** that work. Your test must name the path. Inherit intent. Do not revert hunks. Do not swap designs.
3. Else pick another path from `arch`. Do not busy-loop `look`.

`test_file` means write the test and do not claim it. `new_file` means write the file and do not claim it. `no_test` means the test naming this path is not on disk yet.

If the work is wrong, do not overtake. Leave it.

## Drift

If `write-ok` returns `drift` on a path inside your scope: stop. `wcp look`. Retarget. Never `git reset --hard`, `git checkout --`, or restore to "go first."

## Git

Never commit the board or sqlite. The orchestrator is the only one who runs `git commit`. A worker never commits and never stashes. Stashing on a shared checkout hides another writer's uncommitted files. The orchestrator commits only when `wcp look` shows no live source-file lease. If a lease is live, wait. The work commit is the product paths. The next commit is the `.WCP/issues/` update.

```
.WCP/RUN.md
.WCP/run.sqlite
.WCP/*.sqlite-wal
.WCP/*.sqlite-shm
```

`wcp init` writes those lines and removes a blanket `.WCP/` ignore. Hooks reject any staged `.WCP/` path outside `.WCP/issues/` (socket, lock, pid, log, mode, barrels). Do not store source in `.WCP/`. Do not push. Do not rewind. `WCP_AGENT` is set: hooks will refuse push.

Do not `wcp acquire` an issue file. A ticket lease is the frontmatter on that file. It is not a source-file lock.

## Barrels

If the path matches `.WCP/barrels` (lockfiles, generated clients, root schema): extra-short burst. No thinking on the lease.

## Board

`wcp look` before every write. Edit occupancy only through `wcp`. Do not rewrite `.WCP/RUN.md` by hand (the daemon overwrites it). `RUN.md` is occupancy and `arch` only. Do not put specs, ticket status, or diffs on it.

## End

When the work matches the ticket's `acceptance`, release every source-file lease, then stop. The orchestrator commits and closes the ticket. A worker does not commit. An idle ticket lease is a protocol violation, the same as an idle file line.

## Issues

Committed queue for a solo builder with many agents on one checkout. Do not call Linear, GitHub Issues, or Notion. Assign, claim, and close work only by editing files under `.WCP/issues/`.

### Layout

```
.WCP/issues/
  open/
  in-progress/
  done/
  canceled/
  blocked/
```

One markdown file per issue. Put a stable id in the filename so a status move does not depend on the title: `0123-rotate-refresh-token.md`.

`status` inside the file is the source of truth. The folder is a projection. Update `status`, then move the file. `git mv` once the file is tracked.

### Frontmatter

```yaml
---
id: 0123
title: Rotate refresh token on session mint
status: open | in-progress | done | canceled | blocked
priority: low | normal | high | critical
assignee:          # agent id, empty if unclaimed
lease_expires:     # ISO-8601 UTC, empty if unclaimed
scope:             # what this ticket is allowed to change
acceptance:        # short done definition
files: []          # paths touched while solving
commit:            # hash that closed it, empty until done
reason:            # why it was canceled or blocked; empty unless status is canceled or blocked
---
```

The body under the frontmatter is the spec. Keep it short enough to work from.

### Ticket lease

Claim an open ticket, or an expired `in-progress` ticket:

1. Read the file. If `status` is `in-progress` and `lease_expires` is still in the future, it is held. Pick another.
2. Set `assignee` to your agent id, `status: in-progress`, and `lease_expires` to now + 10 minutes, ISO-8601 UTC (shape `2026-09-24T18:04:00Z`).
3. Write the file, then move it to `in-progress/`.
4. Re-read the file at its new path. If `assignee` is not you, stop. Do not write your id back in.

Renew while you hold it and you are still working: before `lease_expires`, set it to now + 10 minutes again. That edit does not take a source-file lease.

Reclaim when `lease_expires` is past and the assignee is not renewing. The orchestrator does this at the start of a run. Clear `assignee` and `lease_expires`, set `status: open`, move the file to `open/`. Then re-read. If `assignee` is set and `lease_expires` is in the future, that claim stands. Do not clear it again.

Reclaim and claim are idempotent. Only one agent is `assignee`. If two writes race, the last writer re-reads and leaves a single assignee.

### While solving

The orchestrator, or you if no one is assigned, picks one file in `open/`. Skip `blocked/` and `canceled/`. One ticket per agent unless the user says otherwise. Do not claim a directory. Ticket claim is not a file claim.

Use the body as the spec. Implement with the file leases above. Tests first, then a short source lease. Drop the file lease before tests, thinking, or waiting.

Append each touched path to `files` when it lands. Do not put the diff on `RUN.md`.

Two agents do not hold the same ticket. They do not hold the same source file. Different files on one ticket only when `scope` allows it, and each agent still has one live source file.

### Close

When the work matches `acceptance`, the worker releases every source-file lease, leaves the tree dirty, and stops. The worker does not commit and does not stash. The orchestrator:

1. Runs `wcp look`. If any source-file lease is live, wait. Do not commit.
2. Commits the work. `files` already lists the paths. `commit` cannot name a hash that does not exist yet.
3. Writes that hash into `commit`. Sets `status: done`. Clears `assignee` and `lease_expires`. Moves the file to `done/`.
4. Commits that issue-file update. `wcp look` is still empty.

A worker does not commit and does not mark the ticket done. If the orchestrator has not committed, `commit` stays empty and `status` stays `in-progress`.

### Cancel

Cancel when the work will not be done. The holder may cancel a ticket they hold. The orchestrator may cancel an open ticket, or an `in-progress` ticket whose lease is expired. Do not cancel a ticket another agent holds under a live lease.

1. Write `reason`. A canceled ticket with an empty `reason` is not canceled.
2. Set `status: canceled`. Clear `assignee` and `lease_expires`. Leave `commit` empty unless a work hash already exists. Keep `files` if paths already landed.
3. Move the file to `canceled/`. Do not delete it. Do not set `done`.
4. Re-read. One file, one `reason`. If another writer canceled or reclaimed it, leave their newer `assignee` or `reason` in place.
5. The orchestrator commits the issue file, and only when `wcp look` shows no live source-file lease. Until then the file update still stands.

Search `.WCP/issues/canceled/` and `status: canceled` before filing the same work again. Read `reason`. Do not open a second ticket for it unless the user says to revive it. Reviving sets `status: open`, clears the lease, and moves the file to `open/`. Leave `reason` so the earlier cancel stays searchable.

### Block

Block when the work is still wanted and an agent must not claim it yet. The holder may block a ticket they hold. The orchestrator may block an open ticket, or an `in-progress` ticket whose lease is expired. Do not block a ticket another agent holds under a live lease.

1. Write `reason`. A blocked ticket with an empty `reason` is not blocked.
2. Set `status: blocked`. Clear `assignee` and `lease_expires`. Leave `commit` empty unless a work hash already exists. Keep `files` if paths already landed.
3. Move the file to `blocked/`. Do not delete it. Do not set `done`. Do not set `canceled`.
4. Re-read. One file, one `reason`. If another writer blocked, canceled, or reclaimed it, leave their newer `assignee` or `reason` in place.
5. The orchestrator commits the issue file, and only when `wcp look` shows no live source-file lease. Until then the file update still stands.

Do not claim a file in `.WCP/issues/blocked/`. Search that folder and `status: blocked` before filing the same work again. Read `reason`. Unblocking sets `status: open`, clears the lease, and moves the file to `open/`. Leave `reason` so the earlier block stays searchable.

### Orchestrator

Assign work from `.WCP/issues/open/` only. Skip `blocked/` and `canceled/`. No Linear, GitHub Issues, or Notion calls for WCP work. One ticket per agent unless the user says otherwise. Do not claim a directory. Two agents may share a ticket's files only under that ticket's `scope`, and WCP file rules still hold.

### Two clocks

Do not collapse them.

- File overtake is the floor rule in Conflict. A source-file lease is idle when it is `expired` or the pid is dead (default TTL 60 seconds). `overtake` inherits `doing` and `scope`. That clock is not the ticket lease.
- Ticket reclaim is 10 minutes on `lease_expires`.
