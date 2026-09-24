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

Workers do not commit and do not stash. The orchestrator commits the work only when `wcp look` shows no live source-file lease. After that hash exists on local `dev`:

1. Append touched paths to `files`.
2. Write that commit hash into `commit`.
3. Set `status: done`. Clear `assignee` and `lease_expires`. Move the file to `done/`.
4. Commit the issue file. `wcp look` is still empty. Do not stash.

There is no In Review status. `done` is the completion record.

On failure: leave `in-progress` if you still hold it, or `blocked` with `reason` when a human has to answer. Do not set `done` without a hash.

## `/identify`

Claim only the leaf you are about to hand to `/solve`, using the same ticket lease. On abort, reclaim your own unstarted claim back to `open/` (clear `assignee` and `lease_expires`).

## `/prb` and `/yeet`

Do not update an external tracker. The issue file already holds `commit` when `/solve` closed it. Include `.WCP/issues/` in the ship commit.

## `/issues` and `/issue`

File into `open/` unassigned. Never set `in-progress` while filing.
