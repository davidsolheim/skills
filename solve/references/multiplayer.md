# Ticket claim (`/solve` + `/identify`)

The queue is `.wcp/issues/`. Contract: [`../../docs/wcp-queue.md`](../../docs/wcp-queue.md) and skill `water-cooler-protocol`.

Do not call Linear. Do not post claim comments.

Status is the folder: `open`, `in-progress`, `done`, `deployed-dev`, `deployed-main`, `blocked`, `canceled`. An old `in-review/` file is finished writing. Move it to `done/`.

After a file write, mirror to the tracker in `.wcp/tracker.md` when one is named. Skip when it is missing or `none`.

## Unclaimed vs claimed

**Unclaimed:** the file is in `open/`, or it is in `in-progress/` and the agent is gone.

**Claimed by us:** `in-progress/` and `assignee` is this agent.

**Claimed by other:** `in-progress/` and `assignee` is someone else. Skip. Read its `files` and do not take those paths.

## Claim

Before any product edit on that ticket:

1. The leaf is eligible ([`eligibility.md`](eligibility.md)) and unclaimed.
2. Set `assignee`, set `status: in-progress`, and move the file to `in-progress/`.
3. Re-read. If `assignee` is not you, stop. Do not code. Pick another leaf.

`/solve all` and `/solve today`: if every remaining leaf is held by someone else, do not take those. Work unclaimed leaves only.

## Close

The solver does not commit and does not stash. When `acceptance` is met, set `status: done` and move the file to `done/YYYY/MM/DD/` from `created`.

If you edit a file that already has uncommitted changes, read the `in-progress/` or `done/` issue that lists that path and the other paths in its `files`. Keep the behavior its `acceptance` describes.

If anything is still in `in-progress/`, stop. Do not commit. If `in-progress/` is empty and this run has no further task, the orchestrator commits the tree and writes that sha into `dev` on each `done/` issue from that commit.

On failure: leave the file `in-progress` if you still hold it, or move it to `blocked/` with `reason` when a human has to answer.

## `/identify`

Claim only the leaf you are about to hand to `/solve`, the same way. On abort, move your own unstarted claim back to `open/` and clear `assignee`.

## `/prb` and `/yeet`

Do not push from `/solve`. Include `.wcp/issues/` in the ship commit. `deployed-dev/` and `deployed-main/` happen when those commits are actually on those branches.

## `/issues` and `/issue`

File into `open/` with an empty `assignee`. Do not move a new file to `in-progress/` while filing.
