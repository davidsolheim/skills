# RFC: Multiplayer Linear claims (`/solve` + `/prb`)

Status: accepted (David, 2026-08-13) — implement in `/solve` and `/prb` now.
Companion procedure: [`../solve/references/multiplayer-linear.md`](../solve/references/multiplayer-linear.md)

## Why

Grok CLIs, Dev Bots, and humans all hit Linear as **David**. `assignee = me` does not tell who claimed a ticket. Two `/solve all fast` runs double-claimed Designs leaves. `/solve` also marked **Done** on `origin/dev`, so Linear no longer meant shipped.

Linear must be the source of truth for: who claimed what, what is free, what is on `dev`, what is on `main`.

## State machine

| Linear state | Meaning | Who sets it |
| --- | --- | --- |
| Backlog / Todo / Triage | Free. Unassigned. | `/issues` (never pre-assign) |
| **In Progress** | Claimed by one worker. Code in flight. | `/solve` claim (CAS) |
| **In Review** | On `origin/dev`. Not production. | `/solve` after green push to `origin/dev` |
| **Done** | Merged to `main` (and prod ship comment when deploy exists) | `/prb` after merge |
| Blocked | Human/external; not stealable | `/solve` on hard fail needing a person |

If the team has no **In Review**, leave **In Progress** and put `shipped origin/dev @ sha` in the closeout comment. Still do **not** mark Done.

Epic rollup to Done stays allowed only when **all children are terminal** (Done/Canceled/Duplicate).

## Claim protocol (compare-and-swap)

Every Grok session uses the same Linear token. Assignee is not enough.

1. Pick a leaf that is eligible **and unclaimed** (not In Progress unless this session claim comment is already ours).
2. Assign to me if unassigned; set **In Progress**.
3. Post a **claim comment** whose first line is:

`claimed-by: <bot-or-cli> · session <id> · worktree <path-or-cwd> · run <RUN_ID or sequential>`

4. **Immediately re-read** the issue (assignee, state, last claim comments).
5. If another claim comment from a **different** session/bot is newer, or assignee/state does not match: **abort this leaf**, do not code, pick another.
6. Workers in one `/solve all fast` orchestrator share the same `run`. A rival top-level `/solve all` on the same project is forbidden while that run claims are live.

### Steal / stale

Another worker may steal if the latest `claimed-by:` comment is older than **60 minutes** with no later progress comment. Post `stolen-by: …` and proceed. Fast-mode worker timeout (~45–60m) should unclaim (comment `released: stale`) instead of leaving zombie In Progress.

## `/solve all` vs many CLIs

- **One** drain orchestrator per Linear project at a time (`/solve all` / `/solve all fast`).
- Extra CLIs: `/solve TEAM-n` or `/solve N` on **unclaimed** leaves only. Never start a second project-wide drain.
- Before `/solve all`, list In Progress on the project. If foreign `claimed-by:` comments exist, **do not drain** — work around them or stop and report.
- Fast workers under **one** orchestrator remain OK (worktrees + shared run id).

## Git co-work

- Branch from latest `origin/dev` (create `origin/dev` from `main` if missing).
- Before merge/push: `git fetch` + rebase/merge `origin/dev`; if `dev` moved, re-verify.
- Push `origin/dev` with lease-safe history (no force on `dev`).
- Collision on the same files: serialize (Linear still shows who holds the leaf).

## `/prb`

- Comments PR URL on every ship-set issue (In Review / In Progress shipped to `dev`).
- Does **not** mark Done at PR open.
- After **explicit** merge to `main`: mark those issues **Done** + production ship comment (PR, merge SHA, deploy URL when known).
- Do not steal or close someone else In Progress.

## `/issues`

- File **unassigned** Backlog/Todo. Filing is not a claim.

## Bot identity (later)

Claim comments + optional `claimed/<bot>` labels work with one David token. Separate Linear users per Dev Bot are nicer later; not required for this patch.
