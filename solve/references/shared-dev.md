# /solve shared-dev — same-branch parallel

Canonical procedure when `SHARED_DEV` is true. That is the **default** for
every multi-issue `/solve` (`N ≥ 2`, `all`, `/solve today`, scoped drain).
`seq` and opt-in `worktree` disable this file.

Parent skill: [`../SKILL.md`](../SKILL.md)
Eligibility / created-today: [`eligibility.md`](eligibility.md)
Batch guidance: [`batch-guidance.md`](batch-guidance.md)
Claims: [`multiplayer.md`](multiplayer.md)
Git baseline: [`git-dev-workflow.md`](git-dev-workflow.md)
Worktree parallel (opt-in only): [`fast-mode.md`](fast-mode.md)

---

## Purpose

Finish a set of eligible leaves **ASAP** on **one** local `dev` working tree:

1. Inventory (created-today when `SCOPE.kind` is `created`) + S0 guidance.
2. One orchestrator. It **never** implements application source.
3. Spawn a **disjoint-path** ready set in one turn (`solve-implementer`,
   `isolation: none`, same cwd, already on `dev`). Occupancy: [`../../docs/wcp.md`](../../docs/wcp.md).
4. Workers use **wcp exclusive file leases**. No worktrees. No per-issue branches.
5. After **all** live workers finish, each successful leaf is in `done/`.
   The orchestrator commits on `dev` only when `.wcp/issues/in-progress/` is
   empty. No push/PR unless asked. Workers do not commit and do not stash.

Quality that still applies: S0, blocked tickets, wcp leases, one assignee,
runtime proof on the combined tree, drain gate. Quality that this mode
**drops**: worktree isolation, per-issue branches, per-worker verify before
the others finish. File overlap waits for the next wave (occupancy), it does
not become a `blocked/` tracker relation. Overlap is the next wave.

---

## When this file wins

Set `SHARED_DEV = true` (parent parse) for every multi-issue run unless
`FORCE_SEQ` or `WORKTREE_MODE`.

`FORCE_SEQ` → sequential batch loop; this file unused.
`WORKTREE_MODE` (`worktree` / `--worktree`) → [`fast-mode.md`](fast-mode.md).

Do **not** wait for a `fast` flag. Do **not** invent worktrees to “keep merges
clean” — that is the mess this file avoids. Workers write on `dev` under wcp
leases; the orchestrator verifies and commits after they all finish, and
only when `.wcp/issues/in-progress/` is empty. Workers do not stash.

---

## Constants

| Name | Value |
| --- | --- |
| `DEFAULT_CONCURRENCY` | **all ready leaves**, then clamp to `MAX` |
| `MAX_CONCURRENCY` | **32** (host live-child cap). Not the worktree cap of 8 |
| `--concurrency M` | Clamp to `1..32` |
| Isolation | `none` — never `worktree` |
| Branch | local **`dev`** only. No `solve/<RUN_ID>/<ISSUE>` branches |
| Integration | local `dev`. No push/PR unless asked |
| Worker commit | **no** — workers do not commit or stash. Orchestrator commits after combined verify, and only when `.wcp/issues/in-progress/` is empty |
| Soft timeout | ~45–60 min per worker; then kill, fail that leaf, continue others |

`--concurrency` omitted → launch the disjoint-path ready set (≤ 32). Path
overlap and overflow wait for the next wave (still shared-dev, still
combined verify per wave).

---

## Phase extras (after 0–1 + S0)

1. `SHARED_DEV` is already true. `FORCE_SEQ` would have skipped this file.
2. `RUN_ID` = `python3 -c "import uuid; print(uuid.uuid4().hex[:8])"`.
3. Scratch:

```bash
scratch_dir="${TMPDIR:-/tmp}/grok-$(id -u)/solve-shared-${RUN_ID}"
mkdir -p "$scratch_dir/workers" && chmod 700 "${TMPDIR:-/tmp}/grok-$(id -u)" "$scratch_dir"
```

4. `git fetch origin`. Refresh local `main` from `origin/main`. Checkout/create
   local `dev`, merge `origin/main` into it. **Stay on `dev`.** Record
   `dev_tip = $(git rev-parse dev)`.
5. Do **not** create issue branches. Do **not** add worktrees.
6. Trackers: `SOLVED`, `FAILED`, `SKIPPED`, `in_progress` (leaf → subagent id).

If a leaf is `in-progress` and `assignee` is someone else: do not
drain that leaf; skip it.

7. **wcp** ([`../../docs/wcp.md`](../../docs/wcp.md)): read `in-progress/` before the first worker writes. There is no daemon to start or stop.

---

## D1 — Inventory

Same as fast F1, with created-today when `SCOPE.kind` is `created`:

1. Inventory in [`eligibility.md`](eligibility.md).
2. `issue_in_scope` (created-today = local date of `created`).
3. Eligibility 2B / 2D / 2E → **leaves** only. Out-of-scope blockers stay skipped.
4. S0 tags: skip full-obsolete. Guidance still **wins** on platform/stack.
5. Integer `N`: implement until `N` successful **combined-verify commits**.

Do not launch a file in `blocked/` until its blocker is `done` or `canceled`. File overlap is **wcp occupancy**: keep that leaf for the next wave.

---

## D2 — Guidance (required)

S0 must produce `guidance.md` + `graph.json` before the first spawn.

Skip `action=skip` obsolete leaves. Promote migrations in `order_rank` for the
user report. **Launch** unblocked leaves whose **primary write paths**
do not collide with another leaf already in this wave ([`../../docs/wcp.md`](../../docs/wcp.md)).
Hotspot remainder waits for the next wave. One writer if the whole remaining
set shares one primary path.

Direction confidence **low** → stop and ask. Otherwise proceed.

---

## D3 — User report (non-blocking)

```text
Shared-dev plan: K implementable · S skipped · concurrency C · run <RUN_ID>
Scope: created today <YYYY-MM-DD> | <other scope> | none
Launch now: TEAM-…, TEAM-… (disjoint primary paths; wcp leases)
Held for next wave (path overlap): TEAM-…
Delivery: shared local `dev` → done when the writing is finished → commit when in-progress is empty
Worktrees: none
main/prod: /prb (not this run)
```

Then **spawn immediately**. Do not wait for a nod unless direction is low.

---

## D4 — Claim + launch (one turn)

### Claim (orchestrator only)

Before spawn, [`multiplayer.md`](multiplayer.md) for **each**
ready leaf:

1. Unclaimed (`open/`, or `in-progress/` whose agent is gone).
2. Set `assignee`, `status: in-progress`, and move the file to `in-progress/`.
3. Re-read. If `assignee` is not this run, abort that leaf.

Workers move their own ticket to `done/` when acceptance is met.

### Launch rule (non-negotiable)

Wave = not `blocked/`, not skip, **primary write path** disjoint from every
other leaf already in this wave (Occupancy / code map), capped at `CONCURRENCY`.
Overlapping remainder stays queued for D8.

Emit **every** `spawn_subagent` for that wave in the **same** orchestrator
turn. Do not wait for worker A before spawning worker B. Then wait with
`get_command_or_subagent_output` on **all** live ids.

```text
spawn_subagent:
  subagent_type: solve-implementer   # fallback general-purpose
  model: grok-4.6
  isolation: none                    # NEVER worktree
  background: true
  cwd: repo root (same as orchestrator; already on `dev`)
  description: [shared-dev] <ISSUE> <short title>
```

Do not pass a fake `effort:` field. Do not inherit the parent model.

### Worker prompt (required)

```markdown
You are a solve **worker** for a single `.wcp/issues/` leaf on a **shared** local `dev` branch. The issue file is the spec. Do not call Linear.

## Occupancy (wcp) — hard
- Read `$SOLVE_SKILL_DIR/../water-cooler-protocol/SKILL.md` and `$SOLVE_SKILL_DIR/../docs/wcp.md`
- Read `.wcp/issues/in-progress/`. Work around the paths those issues list. There is no `wcp` command.
- Add each path you write to this issue's `files`.
- Never rewind sibling edits. If the file already has uncommitted changes, read the `in-progress/` or `done/` issue that lists that path and the other paths in its `files`. Keep the behavior its `acceptance` describes. Do not commit. Do not push.

## Hard constraints
- Read guidance fully: <GUIDANCE_MD>
- Graph JSON: <GRAPH_JSON> — your id: <ISSUE>
- Guidance wins on stack / rescope
- You share the repo working tree on branch `dev` with sibling workers
- Do not revert, restore, checkout, stash, reset, or overwrite a pre-existing
  file you did not claim. If a file you need already changed, implement your AC on top.
  Do not `git add -A`. Do not `git clean`.
- Do **not** create branches, worktrees, or check out any other branch
- Do **not** commit, stash, merge, push, or open a PR. A stash on this shared tree hides another writer's files
- When acceptance is met: append paths to `files`, set `status: done`, and move the file to `.wcp/issues/done/YYYY/MM/DD/` from `created`. Do not commit. Do not stash.
- Cheap construction: the issue body + custom-implement-instructions.md.
  **No** bundled `/implement` until-zero-nits
- Scope: this leaf only
- Runtime proof for **this leaf** when in-scope (prove-it-works.md). Record
  what you drove in the summary. The orchestrator re-verifies the combined
  tree after every worker finishes — do not wait for siblings
- Write summary to <WORKER_SUMMARY> (paths you changed, decisions, proof notes)
- Do **not** discard unrelated dirty files
```

A successful worker is done when its summary exists and its paths are in the
working tree (or it reports no code change with reason). Origin push is **not**
required. Commit is **not** required.

---

## D5 — Wait, then combined verify (orchestrator)

1. Wait until every live worker in the wave has completed or been killed.
2. Read each `WORKER_SUMMARY`. Map paths → issue ids. Record FAILED workers
   (hard-fail, empty summary + no diff, timeout).
3. `git status` / `git diff`. Unrelated pre-existing dirty files stay unstaged.
4. **Combined verify** on the current `dev` working tree — **one** pass for the
   wave, not per worker before the others finish:

   Authority: [`../../docs/prove-it-works.md`](../../docs/prove-it-works.md).

   - Matrix from `AGENTS.md` / README for touched packages
   - Runtime proof for every in-scope class touched by **any** leaf in the wave
   - Per-leaf AC from summaries + remaining diff

5. Matrix green is necessary, not sufficient, for in-scope work.
6. On verify fail: **resume** the implementer(s) whose paths/AC failed (or spawn
   a new `solve-implementer` for that id). Do **not** edit application source
   yourself. Re-run combined verify. Do not move a leaf to `done/` that still fails.
7. Independent failures: leave that leaf `in-progress`/`blocked` with `reason`;
   **continue** other leaves. Cascade-skip only files in `blocked/` with
   `reason: blocked by <id>`, not file-overlap neighbors.

**Inner review:** if **any** leaf in the wave is heavy/critical (or `--effort`
asked for a reviewer), spawn **one** `solve-reviewer` on the combined diff after
verify, bugs only. Nits do not block. Resume implementers once for open bugs.

---

## D6 — Confirm `done/`

Workers do not commit. After combined verify passes, each successful leaf is in `done/`. If a successful worker left the file `in-progress` and acceptance is met, move it to `done/`.

Do not commit while any issue is still in `in-progress/`.

## D6b — Commit on local `dev` (orchestrator)

1. Read `in-progress/`. If any issue is there, wait. Do not commit. Do not stash.
2. Stage only this wave’s finished product paths. Leave unrelated dirty files unstaged. Leave the issue files for D7. Never secrets / `.env` / `.wcp/RUN.md` / `.wcp/run.sqlite` / sqlite wal/shm. Never `git add -A`.
3. Commit the work. `commit` on the issue file cannot name a hash that does not exist yet.

Prefer **one commit per issue** when worker summaries partition the diff
(`git add` only that issue’s paths; subject `0123: short imperative`).
If paths overlap and cannot be split, **one** commit listing every successful
id in the subject/body.

```text
solve: 0123, 0124, 0125

- 0123: …
- 0124: …
```

HEREDOC. `dev` already has the work — there is **no** merge step and **no**
branch delete. Confirm `git log -1` is on `dev`.

---

## D7 — Write the hash

The reviewer already set `done`. `in-progress/` is still empty. For each leaf whose paths landed in the work commit:

1. Append paths to `files` if they are not already listed. Write the work-commit hash into `dev`.
2. Unblock any `blocked/` ticket whose `reason` names this id.

Then commit those issue-file updates. If `in-progress/` is not empty, wait. Do not stash.

Failed leaves: leave `in-progress` or set `blocked` with `reason`. Do not take a live lease held by someone else. Those edits ride in the same issue-file commit when the board is empty.

---

## D8 — Refill + drain gate

After the wave is committed on local `dev`:

1. Re-read `.wcp/issues/` (created-today filter still on when `SCOPE.kind` is `created`).
2. Newly unblocked in-scope leaves (blocker now `on_dev`) plus occupancy-held
   leaves (primary path now free) enter the next disjoint-path wave on the
   same `dev`. Spawn them the same way (one turn, wcp leases).
3. Combined verify, then commit only when `.wcp/issues/in-progress/` is empty.
4. `SELECTION_PIN` / `SCOPE`: do not append outside the pin or `issue_in_scope`.

### Drain gate (`all`, including `/solve today`)

Fresh read of `.wcp/issues/` + scope filter + eligibility. If any implementable
**unclaimed** leaf remains **in the active set** (today’s window when created
scope) → resume D4. Do not Phase 9. `/solve today` may finish while older
eligible issues still exist — that is correct.

---

## Phase 9 — Shared-dev summary

**`all` / today:** only after drain gate.

```markdown
**Batch:** solved K of <N|all|today> · failed F · skipped S
**Mode:** /solve <today|N|all> · shared-dev · concurrency C · run <RUN_ID>
**Scope:** created today <YYYY-MM-DD> | <area> | none
**Delivery:** shared local `dev` (no worktrees) → combined verify → commit
**local `dev`:** <sha>
**Drain (all/today):** verified — no eligible unblocked unclaimed leaves [created today <date>]

### Solved (`done/` on local `dev`)
1. TEAM-123 — local `dev` <sha>

### Failed
- [TEAM-125](url) — reason

**main/prod:** not shipped — run `/prb` when ready
```

---

## Anti-patterns (shared-dev)

- Using `isolation: worktree` or `solve/<RUN_ID>/<ISSUE>` branches in this mode
- Verifying worker A (and merging) before spawning or waiting for worker B
- Waiting for an explicit `fast` flag
- Treating file overlap as a `blocked/` relation (it is wcp occupancy: next wave)
- Launching two workers on the same primary write path in one wave
- Worker edits a path another `in-progress/` issue already lists
- Holding a wcp lease through tests or combined verify
- Rewinding sibling hunks (`git checkout --`, reset, restore) to “go first”
- Launching a `blocked by <id>` dependent before the blocker is `on_dev`
- Worker commit, stash, merge, or push
- Orchestrator commit or stash while an issue is in `in-progress/`
- Orchestrator implementing application source
- `git add -A` / `git reset --hard` / `git checkout --` that discards siblings
  or unrelated dirty files
- Treating `/solve today` as a project-wide `/solve all`
- Treating `today` as an area named Today
- UTC-only `createdAt` (misses local-morning issues)
- Capping shared-dev at worktree concurrency **8** when more leaves are ready
- Stashing, worktree-merging, or creating issue branches “to keep dev clean”
- Phase 9 while created-today eligible leaves remain
- Setting Notion **done** from `/solve`
- Skipping S0 or combined runtime proof “to go faster”
