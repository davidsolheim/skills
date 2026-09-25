# WCP for these skills

Occupancy for many coding agents on one local `dev` checkout.
Spec: [watercoolerprotocol.com](https://watercoolerprotocol.com) /
[PROTOCOL.md](https://github.com/davidsolheim/water-cooler-protocol/blob/dev/PROTOCOL.md).
Player verbs: skill `water-cooler-protocol`.

This file is how every skill that edits a checkout uses that protocol:
`/solve`, `/issues`, `/issue`, `/identify`, `/start`, `/prb`, `/yeet`,
`/project-review`, `/walk`, `/tidy`, `/human-copy`, `/vercel-flags`,
TypeSafe, composition refactors, `/implement`, and `/execute-plan`.
Do not fork the verbs here.

## What occupancy is

A lease is an edit burst on **one pre-existing path**. One agent per path.
One live path per agent. Write the test first; do not claim the test file.
Write a new file directly; do not claim it. Claim only a file that was already
in the tree when the run started, and only after a test names that path.
Release the instant bytes are on disk — before tests, thinking, or waiting.
Drift is other workers. Never rewind the tree.

A ticket that must not be claimed yet moves to `.WCP/issues/blocked/` with `reason` set. Sharing a source file does not block a ticket. File overlap waits for the next wave.

## Who loads it

| Actor | Duty |
|-------|------|
| `/solve` orchestrator | Start the run (`wcp init` / `wcp start --arch`). Do not assign ids. Prefer a disjoint-path wave. Combined verify, then commit on `dev` only when `wcp look` shows no live source-file lease. Never implement app source. Never push. Never stash a live burst. |
| `/solve` implementer | Player skill. `wcp name <id>`, then export `WCP_AGENT` and `WCP_NAME_TOKEN`. Write the test (no claim), write new files (no claim), then look → acquire --test → write-ok → re-read disk → edit → release for files that already existed. Do not commit. Do not stash. |
| `/prb` fixer | Same player verbs on local `dev`. |
| `/issue` `/issues` `/project-review` `/walk` `/tidy` `/identify` `/stat` | The local queue is `.WCP/issues/` ([`wcp-queue.md`](wcp-queue.md)). Notion status is [`notion-issues.md`](notion-issues.md). Fill **Occupancy (WCP)** on each leaf. Do not call Linear. |
| `/prb` `/yeet` push | Human export to `origin/dev`. `wcp look` before commit and before push. Unset `WCP_AGENT` and `WCP_NAME_TOKEN` before `git push`. |
| Any other builder on this checkout (`/human-copy`, `/vercel-flags`, `/start`, TypeSafe, composition refactors, `/implement`, `/check-work` fixes) | Same player turn as an implementer. Release before tests or the next thought. Do not commit and do not stash. The session that owns the checkout commits when `wcp look` shows no live source-file lease. |
| `/execute-plan` | Implementers stay in their own worktrees. Do not put those worktrees on the shared checkout's board. Orchestrator edits on the shared tree use the player turn. |

Worktree `/solve` (`worktree` / `--worktree`) is a different isolation. WCP is
one live checkout. Do not run one WCP board across worktrees.

## Binary

```bash
WCP_BIN="$(command -v wcp || true)"
if [ -z "$WCP_BIN" ] && [ -x "$HOME/src/water-cooler-protocol/dist/wcp" ]; then
  WCP_BIN="$HOME/src/water-cooler-protocol/dist/wcp"
fi
```

If `WCP_BIN` is missing: honor-system still applies (no rewind, no agent push,
no overwrite of a path another worker holds). Report once. Do not abort `/solve`.

## Orchestrator: start a run

After checkout is on local `dev`, before the first implementer write:

```bash
"$WCP_BIN" look --json
# no run → init
"$WCP_BIN" init --arch "<aim>" --branch dev
# run exists → keep it; do not wipe live rows
```

`arch` is the session aim: batch-guidance canonical platforms + this `/solve`
run id, or the single-issue title. The orchestrator may `init` / `start`.
Workers never `set-arch` or `stop`.

Then read `.WCP/issues/open/` and `.WCP/issues/in-progress/`. Reclaim expired
tickets (player skill, Issues). Do not copy the backlog onto `RUN.md`. If the
queue directories are missing and this run needs a queue, create `open/`,
`in-progress/`, `in-review/`, `done/`, `canceled/`, and `blocked/`, and one issue from `arch` only.
Search `.WCP/issues/canceled/` and `.WCP/issues/blocked/` before filing the same work again. Cancel and block in
the issue file with `reason` set (player skill, Cancel and Block). Do not claim a blocked ticket.

Assign WCP work from `.WCP/issues/open/` only. One ticket per agent unless the user
says otherwise. Do not claim a directory. A ticket lease is not a source-file
lease. Do not call Linear or GitHub Issues. After an issue file changes
status, update Notion ([`notion-issues.md`](notion-issues.md)). Do not call
Notion during a source-file lease. Skill filing, solving,
status, tidy, and ship all use this queue ([`wcp-queue.md`](wcp-queue.md)).

Do not `wcp stop` at the end of `/solve` (other writers may still be on the
tree). `/prb` / `/yeet` may stop a run only when this checkout is the ship
and `look` shows no live leases.

## Agent ids

The worker names itself. `WCP_AGENT` matches `^[a-z0-9][a-z0-9._:-]{0,63}$`.

```bash
wcp name issue-123 --json
export WCP_AGENT=<agent>
export WCP_NAME_TOKEN=<token>
```

Prefer the issue id lowercased. On `name_taken`, use `issue-123-b`. A `/prb` fixer prefers `prb-fix`. Do not rename after the token is exported.

## Implementer turn

Load skill `water-cooler-protocol`. Then:

```
wcp name <id> --json
# export WCP_AGENT and WCP_NAME_TOKEN
wcp look --json
# queue: read .WCP/issues/open and in-progress; claim in the file (skill Issues). Do not acquire it.
# tests first — write the test, do not acquire it
# // WCP <id>: <existing-path> <what it proves> (<arch>)
# new file: write it, no acquire
wcp acquire --path <existing-file> --test <test-file> --doing "<burst>" --scope "<symbol>" --json
wcp write-ok --path <existing-file> --json
# re-read that file from disk, edit, flush
wcp release --json
# run YOUR tests only
```

Test line: `// WCP <id>: <path> <what it proves> (<arch>)`
(`# WCP` / `-- WCP` in other languages). The path is the existing file you will claim. Prefer a per-slice test file.

Conflict and `write-ok` errors: skill `water-cooler-protocol` +
`references/playbook.md`. Overtake only to finish idle work. Never
`git reset --hard`, `git checkout --`, or restore a sibling’s file.

## Parallel `/solve` (shared-dev)

Workers share one `dev` tree (`isolation: none`). Occupancy is WCP.

1. Inventory primary write paths from each leaf’s Occupancy / code map.
2. Launch a **disjoint-path** ready set up to `CONCURRENCY` (host cap 32;
   WCP operating point is about 8–20 writers on disjoint files).
3. A leaf whose primary path collides with a live wave-mate waits for the
   next wave. That is occupancy, not Linear `blockedBy`.
4. If the whole remaining set is one hotspot, launch **one** writer.
5. Workers still look/acquire/write-ok/release on pre-existing files. Tests and new files are written with no claim. A surprise collision follows
   the player conflict order (retarget / overtake idle / pick another path).
6. After the wave, orchestrator combined-verifies (full green is this gate,
   not a held lease). Workers have released every source-file lease.
   `wcp look` must show none live. Then the orchestrator commits on `dev`.
   Do not stash to clear the index. Still no push.

## Intake (tickets)

Every create leaf includes **Occupancy (WCP)** from
[`../issue/references/issue-body-template.md`](../issue/references/issue-body-template.md).

Split so two `/solve` workers can hold different primary paths. File overlap
is occupancy, not a reason to merge tickets. `blockedBy` only when B’s AC is
impossible until A lands.

## Human export (`/prb`, `/yeet`)

Agents do not push `origin/dev`. These skills **are** the export.

1. `wcp look --json` — if live leases remain, wait or report; do not push
   through a burst.
2. Fixer writes use player verbs. It names itself, preferring `prb-fix`.
3. `wcp look --json` before an auto-commit as well. If a source-file lease
   is live, wait. Do not commit that burst and do not stash it.
4. Before every `git push` (`origin dev` or any other remote): `unset WCP_AGENT WCP_NAME_TOKEN`
   (hooks refuse push while `WCP_AGENT` is set). Commit `.WCP/issues/`. Do not commit `.WCP/RUN.md`, `.WCP/run.sqlite`, or sqlite wal/shm.
