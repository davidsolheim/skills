# Ticket claim (`/solve` + `/identify`)

The queue is `.WCP/issues/`. Contract: [`../../docs/wcp-queue.md`](../../docs/wcp-queue.md). Verbs: skill `water-cooler-protocol`, section Issues.

Do not call Linear. Do not post `claimed-by` comments.

## Unclaimed vs claimed

**Unclaimed:** file is in `open/`, or `in-progress/` with `lease_expires` in the past.

**Claimed by us:** `in-progress/`, `assignee` is this agent, `lease_expires` is in the future.

**Claimed by other:** `in-progress/`, `assignee` is someone else, `lease_expires` is in the future. Skip. Do not implement.

## Claim

Before any product edit on that ticket:

1. The leaf is eligible ([`eligibility.md`](eligibility.md)) and unclaimed.
2. Claim with the player skill: set `assignee`, `status: in-progress`, `lease_expires` to now + 10 minutes UTC, move to `in-progress/`.
3. Re-read. If `assignee` is not you, abort. Do not code. Pick another leaf.
4. Renew before `lease_expires` while you are still writing the issue file or about to write source. Drop the source-file lease before tests. The ticket lease is separate.

`/solve all`: if every remaining leaf is held by someone else under a live lease, do not drain those. Work unclaimed leaves only.

## Close

The solver does not commit and does not stash. When acceptance is met, the solver sets `status: in-review`, clears `lease_expires`, and moves the file to `in-review/`.

The orchestrator launches one reviewer per file in `in-review/`. The reviewer checks security, accessibility, functionality, and aesthetics, fixes failures under a file lease, and sets `status: done`. The reviewer does not commit.

After that reviewer has exited and `wcp look` shows no live source-file lease, the orchestrator commits the work, writes that hash into `commit`, and commits the issue file.

On failure before review: leave `in-progress` if you still hold it, or `blocked` with `reason` when a human has to answer. The reviewer is the one who sets `done`.

## `/identify`

Claim only the leaf you are about to hand to `/solve`, using the same ticket lease. On abort, reclaim your own unstarted claim back to `open/` (clear `assignee` and `lease_expires`).

## `/prb` and `/yeet`

Do not update an external tracker. The issue file already holds `commit` when `/solve` closed it. Include `.WCP/issues/` in the ship commit.

## `/issues` and `/issue`

File into `open/` unassigned. Never set `in-progress` while filing.
