# WCP in these skills

Occupancy for many coding agents on one local `dev` checkout.
Spec: [watercoolerprotocol.com](https://watercoolerprotocol.com).
Player verbs: skill `water-cooler-protocol`.

This file is how `/solve`, `/issues`, `/issue`, `/identify`, `/start`, `/prb`,
and `/yeet` use that protocol. Do not fork the verbs here.

## What occupancy is

A lease is an edit burst on **one path**. One agent per path. One live path per
agent. Release the instant bytes are on disk — before tests, thinking, or
waiting. Drift is other workers. Never rewind the tree.

Linear `blockedBy` is product dependency. WCP is file occupancy. Sharing a file
does not mint `blockedBy`.

## Who loads it

| Actor | Duty |
|-------|------|
| `/solve` orchestrator | Start the run (`wcp init` / `wcp start --arch`). Assign ids. Prefer disjoint primary paths when several writers share one tree. Combined verify + commit. Never implement app source. Never push. |
| `/solve` implementer | Player skill. `export WCP_AGENT`. look → acquire → write-ok → re-read disk → edit → release. Tests first. |
| `/prb` fixer | Same player verbs on local `dev`. |
| `/issue` `/issues` `/project-review` `/walk` `/tidy` `/identify` upgrade | Fill **Occupancy (WCP)** on each leaf. Prefer disjoint primary write paths. No app writes. |
| `/prb` `/yeet` push | Human export to `origin/dev`. Unset `WCP_AGENT` before `git push`. |

Worktree `/solve` (`fast` / `--fast` in this pack) is a different isolation.
WCP is one live checkout. Do not run one WCP board across worktrees.

## Binary

```bash
WCP_BIN="$(command -v wcp || true)"
```

If missing: put `dist/wcp` from
[water-cooler-protocol](https://github.com/davidsolheim/water-cooler-protocol)
on `PATH`, or honor-system (no rewind, no agent push, no overwrite of a path
another worker holds). Report once. Do not abort `/solve`.

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

Do not `wcp stop` at the end of `/solve` (other writers may still be on the
tree). `/prb` / `/yeet` may stop a run only when this checkout is the ship
and `look` shows no live leases.

## Agent ids

`WCP_AGENT` matches `^[a-z0-9][a-z0-9._:-]{0,63}$`.

| Role | Id |
|------|----|
| Implementer for `TEAM-331` | `team-331` |
| Collision on that id | `team-331-b` |
| `/prb` fixer | `prb-fix` |
| Sequential `/solve 1` | same issue id |

Prompt: `you are <id>`. Worker exports it and does not invent a second id.

## Implementer turn

Load skill `water-cooler-protocol`. Then:

```
wcp look --json
# tests first — acquire the test path, write, release
wcp acquire --path <file> --doing "<burst>" --scope "<symbol>" --json
wcp write-ok --path <file> --json
# re-read that file from disk, edit, flush
wcp release --json
# run YOUR tests only
```

First line of a new test: `// WCP <id>: <what it proves> (<arch>)`
(`# WCP` / `-- WCP` in other languages). Prefer a per-slice test file.

Conflict and `write-ok` errors: skill `water-cooler-protocol` +
`references/playbook.md`. Overtake only to finish idle work. Never
`git reset --hard`, `git checkout --`, or restore a sibling’s file.

## Parallel writers on one tree

When several `/solve` workers share one `dev` working tree (`isolation: none`):

1. Inventory primary write paths from each leaf’s Occupancy / code map.
2. Launch a **disjoint-path** ready set.
3. A leaf whose primary path collides with a live wave-mate waits for the
   next wave. That is occupancy, not Linear `blockedBy`.
4. If the whole remaining set is one hotspot, launch **one** writer.
5. Workers still look/acquire/write-ok/release.
6. After the wave, orchestrator verifies (full green is this gate, not a held
   lease). Then commit on `dev`. Still no push.

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
2. Fixer writes use player verbs (`WCP_AGENT=prb-fix`).
3. Before `git push origin dev`: `unset WCP_AGENT` (hooks refuse push while
   it is set). Do not commit `.WCP/`.
