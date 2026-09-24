# /solve parallel — Orchestrator protocol

Canonical procedure when `FAST_MODE` is true. That is **opt-in only**
(`worktree` / `--worktree`). Default parallel is [`shared-dev.md`](shared-dev.md)
(same local `dev`, orchestrator verifies then commits). Single-issue `/solve`
and explicit `seq` / `--sequential` ignore this file.

If `SHARED_DEV` is true (the default for batches), **stop** and follow
[`shared-dev.md`](shared-dev.md). This file is **worktree** isolation.

Parent skill: [`../SKILL.md`](../SKILL.md)
Batch guidance: [`batch-guidance.md`](batch-guidance.md)
Claims / Linear states: [`multiplayer-linear.md`](multiplayer-linear.md)
Git baseline: [`git-dev-workflow.md`](git-dev-workflow.md)

---

## Purpose

Run many eligible Linear leaves **in parallel** without losing quality, work,
or double-claiming.

1. Inventory + batch guidance (IA, supersession, conflict waves).
2. **One** orchestrator per Linear project (the parent `/solve` session). Extra
   CLIs only take unclaimed leaves; they never start a second drain.
3. Workers implement in **worktrees** (`solve-implementer`, `isolation: worktree`).
4. Orchestrator merges ready issue branches into **local `dev`** in `merge_order`.
   Linear **In Review** after that merge. No PR unless the user asked.
5. Next wave **rebases on the new local `dev`**. `/prb` still owns `main`.

Linear is the claim board (`claimed-by:` CAS). Git branches are backup + merge
surface — **never named after an agent**.

Quality is unchanged from sequential: S0, hard-dep/conflict serialization,
runtime proof, per-issue inner review, drain gate, CAS.

---

## Constants

| Name | Value |
| --- | --- |
| `DEFAULT_CONCURRENCY` | `8` (`all` / scoped drain); then capped by ready independent leaves |
| `MAX_CONCURRENCY` | `8` |
| Integer-mode concurrency | `min(N, MAX_CONCURRENCY)` unless `--concurrency` set |
| `--concurrency M` | Override width; clamp to `1..MAX_CONCURRENCY` |
| Soft worker timeout | ~45–60 minutes; then kill, fail, cleanup, unclaim |
| Issue branch | `solve/<RUN_ID>/<ISSUE>` (e.g. `solve/a1b2c3d4/KEC-780`) |
| Integration | **local `dev`** (lowercase). No push/PR unless the user asked |
| `main` / prod | never under this protocol. That is `/prb` |

### Count vs concurrency

| Invocation | Issues this run | Max concurrent workers |
| --- | --- | --- |
| `/solve` / `/solve 1` | 1 | 1 (this file unused) |
| `/solve 5` | up to **5 issues** | up to `min(5, 8)` |
| `/solve all` | **all** eligible in the active set (no soft cap) | `DEFAULT_CONCURRENCY` (8) |
| `/solve all --concurrency 5` | **all** eligible in the active set | 5 |
| `/solve <scope>` | **all** in `SCOPE` (milestone/label/area) | `DEFAULT_CONCURRENCY` (8) |
| any + `seq` | same count | 1 (sequential loop; this file unused) |

Default concurrency **8** and inner-review yes/no are **not** issue counts.
Inner review comes from [`../../docs/intensity.md`](../../docs/intensity.md)
per issue (or `--effort`, capped at 1). `fast` / `--fast` is a no-op.

---

## Branch rules

- **Do not** name branches after the bot/CLI (`grok-3`, `codex`, `cursor`).
- Issue branch is always `solve/<RUN_ID>/<ISSUE>` with the Linear identifier
  (uppercase as Linear shows it).
- Never force-push `dev` or `main`.
- Do **not** `git push` or open a PR unless the user asked. Worktree HEAD is
  enough for the orchestrator to merge to local `dev`.

---

## Phase 0 extras (parallel)

After normal Phase 0–1 bootstrap:

1. `FAST_MODE` is already true (parent skill). `FORCE_SEQ` would have skipped this file.
2. Resolve `CONCURRENCY`.
3. `RUN_ID` = `python3 -c "import uuid; print(uuid.uuid4().hex[:8])"`.
4. Scratch dir:

```bash
scratch_dir="${TMPDIR:-/tmp}/grok-$(id -u)/solve-fast-${RUN_ID}"
mkdir -p "$scratch_dir/workers" && chmod 700 "${TMPDIR:-/tmp}/grok-$(id -u)" "$scratch_dir"
```

5. Paths: `GUIDANCE_MD`, `INVENTORY_JSON`, `GRAPH_JSON`, `STATE_JSON`,
   `WORKER_SUMMARY(id)` under scratch.
6. `git fetch origin`. Refresh local `main` from `origin/main`. Checkout/create
   local `dev`, merge `origin/main` into it. Record
   `dev_tip = $(git rev-parse dev)`. Do **not** push `origin/dev` unless the
   user asked.
7. Trackers: `SOLVED`, `FAILED`, `SKIPPED`, `WORKTREES_CLEANED`,
   `BRANCHES_DELETED`, `in_progress`.

If In Progress on this project already has **foreign live `claimed-by:`**
comments: **do not drain**. Report and stop, or work around with `/solve N`
on unclaimed leaves only.

---

## F1 — Full eligible inventory

Pre-scan via **Phase S0 / batch-guidance.md**:

1. Linear inventory fetch in [`eligibility.md`](eligibility.md) (team + project
   + state per actionable status; slim fields; page each state).
2. If `SCOPE` is set, keep `issue_in_scope` only (Scope filter in the same file).
3. Eligibility (2B), blocked (2D), epic expand (2E) → **leaves** only.
   Canonical: [`eligibility.md`](eligibility.md). Out-of-scope blockers stay skipped.
4. Tag platforms, class, supersedes; skip full-obsolete.
5. Order by `order_rank` (migrations first; lowest number is tie-break).
6. Integer `N`: inventory all for guidance; implement until `N` successful
   **merges to local `dev`**.
7. Do **not** comment ordinary scan skips. Do comment once when canceling a
   full-obsolete supersede.

After each wave lands on local `dev`, **re-scan** Linear, patch guidance,
append waves. Do not freeze F1.

---

## F2 — Batch guidance + waves

Same graph fields as sequential S0 (`hard_deps`, `conflict_zones`,
`primary_paths`, supersession).

**Wave assignment:**

```text
level(issue) = 0 if no unfinished hard_deps and not waiting on conflict serialize
level(issue) = max(level(dep) for dep in hard_deps ∪ serialize_predecessors) + 1
```

Within a wave, `merge_order` = `order_rank`, then lowest issue number.

Waves are **heuristics**. Overlapping `primary_paths` serialize. A
“non-overlapping” wave can still conflict at merge time — rebase/resolve or
fail that leaf; do not force `dev`.

Do **not** start workers until `GUIDANCE_MD` and `GRAPH_JSON` exist and
direction confidence is not blocking **low**.

---

## F3 — User report (non-blocking)

```text
Parallel plan: K implementable · S skipped · W waves · concurrency C · run <RUN_ID>
Wave 0: TEAM-123 (migration)
Wave 1: TEAM-80, TEAM-91 (independent)
Skip: TEAM-67 (abandoned platform)
Delivery: worktrees → orchestrator merge to local dev → In Review
main/prod: /prb (not this run)
```

Proceed unless direction confidence is low or the user asked for dry-run.

---

## F4 — Worker loop

### Claim (orchestrator only, CAS)

Before spawn, follow [`multiplayer-linear.md`](multiplayer-linear.md):

1. Confirm unclaimed (no foreign live `claimed-by:`).
2. Assign to me if unassigned; set **In Progress**.
3. Claim comment, first line:
   `claimed-by: solve-fast · session <id> · worktree <path> · run <RUN_ID> · branch solve/<RUN_ID>/<ISSUE>`
   Then: wave, plan, verify, delivery = local `dev` then In Review (not Done).
4. **Re-read immediately.** If another run’s claim is newer, abort this leaf.
5. Graph status `claimed` → `implementing`.

Workers **must not** set Linear state.

### Branch + worktree

- `wave_base_sha` = `git rev-parse dev` at wave start.
- `issue_branch = solve/<RUN_ID>/<ISSUE>`
- `git branch <issue_branch> <wave_base_sha>` then spawn `isolation: worktree`
  on that branch.

Worker spawn (required): `subagent_type: solve-implementer` (fallback
`general-purpose`), `isolation: worktree`, **`model: grok-4.6`**
([`../../docs/grok-models.md`](../../docs/grok-models.md)), `background: true`.
Do not inherit the parent. Do not pass a fake `effort:` field.

### Launch rule (non-negotiable)

When the ready set is non-empty, **emit every `spawn_subagent` for that set
(up to `CONCURRENCY`) in the same orchestrator turn.** Do not wait for worker
A to finish before spawning worker B. Then wait with
`get_command_or_subagent_output` on the live ids. Inner `solve-reviewer`
(heavy/critical only) runs **inside** that worker’s worktree after its
implementer, not as a second project-wide swarm.

### Worker prompt (required)

```markdown
You are a solve **worker** for a single Linear leaf.

## Hard constraints
- Read guidance fully: <GUIDANCE_MD>
- Graph JSON: <GRAPH_JSON> — your id: <ISSUE>
- Guidance wins on stack / rescope
- Worktree only; branch: solve/<RUN_ID>/<ISSUE>
- Base is this wave’s local `dev` tip. Do not merge other issues.
- Cheap construction: apply the Linear contract + custom-implement-instructions.md. **No** bundled `/implement` until-zero-nits.
- Inner review: <none | bugs-only from intensity.md / --effort, capped at 1>
- Verify per AGENTS / issue AC **and** [`../../docs/prove-it-works.md`](../../docs/prove-it-works.md) (runtime proof when in-scope). Matrix green is not enough.
- Do not commit. Do not stash. Leave the worktree dirty.
- Write summary to <WORKER_SUMMARY>
- Do **not** merge `dev` or `main`, do **not** open a PR, do **not** push
- Do **not** discard unrelated dirty files
- Scope: this leaf only
```

A successful worker is done when its summary exists and its paths are in the
worktree. The orchestrator commits on the issue branch after the worker exits.
Origin push is **not** required.

### Ready-queue

```text
while work remains:
  ready = pending/claimed leaves whose hard_deps and conflict predecessors
          are **merged to local `dev`**, under concurrency budget
  launch ALL ready workers in one turn until CONCURRENCY
  on worker success → status ready_to_merge
  on worker fail → cleanup local WT; comment Linear; leave In Progress/Blocked;
                   cascade-skip dependents; continue independents
  when the current wave’s launched issues are all ready_to_merge or failed → F5
```

Never exceed `CONCURRENCY` live worktrees. Prefer clean-then-launch.

---

## F5 — Merge wave into local `dev` (orchestrator only)

Workers never merge `dev` and never commit. The orchestrator does, after the wave’s launched
issues are `ready_to_merge` or `failed`, and at least one is `ready_to_merge`.

### Commit in the worktree

The worker left the worktree dirty. After that worker has exited, commit on the issue branch inside the worktree. Stage only that leaf’s paths. Do not stash. Then merge that commit into local `dev` in the main workspace. The main workspace waits until its `wcp look` shows no live source-file lease. Do not stash the main workspace to make the merge.

### Merge

```bash
git checkout dev
# merge_order among ready_to_merge only
for ISSUE in $MERGE_ORDER; do
  git merge --no-ff solve/<RUN_ID>/$ISSUE \
    -m "Merge solve/<RUN_ID>/$ISSUE into dev"
  # conflict: abort that merge, mark ISSUE failed, Linear comment, continue others
done
```

If a worktree isolation branch is not in the main repo yet, fetch it first
(`git fetch <worktree_path> HEAD --no-tags`) per
[`git-dev-workflow.md`](git-dev-workflow.md).

Re-verify on `dev` if the merge was not a clean ff of an already-verified
issue branch (conflict resolution, stacked merges). Do not In Review a leaf
whose post-merge `dev` failed required checks or in-scope runtime proof.

If **every** merge in the wave conflicts, stop the wave, report, do not
pretend `dev` moved.

### Linear closeout (after local `dev` has the leaf)

For each issue that landed:

1. Completion comment: issue branch, local `dev` SHA, matrix + runtime-proof
   evidence ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)).
2. **In Review** (or stay In Progress if the team has no In Review). **Never Done.**
3. Epic rollup only when all children are terminal.

Issues that failed merge stay In Progress/Blocked with a failure comment.
Do not steal foreign claims.

### Cleanup (mandatory after wave merge or fail)

```text
1. kill worker if still alive
2. grok worktree rm --force <worktree_path>
3. delete **local** issue branch (never delete dev or main)
4. End of run: 0 leftover solve-fast worktrees for this RUN_ID
```

Do not remove unrelated user worktrees.

---

## F6 — Refill

After a wave is on local `dev`:

1. Next wave base is current `dev`.
2. Re-list Linear (all pages). Patch guidance + graph.
3. Continue until integer `N` met, `all` drain gate passes, or nothing eligible remains.

If `SELECTION_PIN` is set (Identify), **do not** append leaves outside the pin.
See [`eligibility.md`](eligibility.md) (Selection pin). If `SCOPE` is set,
**do not** append leaves outside `issue_in_scope` (Scope filter).

### Drain gate (`all` only)

Fresh Linear inventory fetch + scope filter + eligibility. If any implementable
**unclaimed** leaf remains **in the active set** → resume F4. Do not Phase 9.
Scoped runs may finish while the rest of the project still has eligible leaves.

---

## Phase 9 — Parallel summary

**`all`:** only after drain gate.

```markdown
**Batch:** solved K of <N|all> · failed F · skipped S
**Mode:** /solve <N|all> · concurrency C · run <RUN_ID>
**Delivery:** worktrees → local `dev`
**local `dev`:** <sha>
**Cleanup:** worktrees 0 remaining
**Scope:** <milestone Name | label X | area "…" | none>
**Drain (all):** verified — no eligible unblocked unclaimed leaves [in this scope]

### Solved (In Review on local `dev`)
1. [TEAM-123](url) — local `dev` <sha>

### Failed
- [TEAM-125](url) — reason

**main/prod:** not shipped — run `/prb` when ready
```

---

## Anti-patterns (parallel-specific)

- Using this file for `/solve today` or same-branch / ignore-each-other requests (`SHARED_DEV` → [`shared-dev.md`](shared-dev.md))
- Waiting for an explicit `fast` flag before using this file on `/solve N` / `/solve all`
- Spawning ready workers serially (wait for A, then spawn B)
- Second `/solve all` drain on a project with foreign live claims
- Naming branches after a bot/CLI
- Starting workers before guidance.md + graph.json
- Treating “non-overlapping” as a guarantee (skip rebase when `dev` moved)
- Worker merging `dev` or `main`, opening a PR, or setting Linear Done
- Merging a dependent wave before hard deps are on **local `dev`**
- `gh pr merge` to **main** (or to `dev` unless the user asked)
- Marking Linear **Done** from `/solve`
- Treating worker typecheck/tests as runtime proof for in-scope UI/auth/billing/API/schema/shared-helper leaves
- Hard-stopping the whole run when one independent leaf fails
- Stopping `/solve all` after ~5 or after wave 0 without refill + drain gate
- Picking a leaf **outside** `SCOPE` during a scoped drain
- Exceeding `MAX_CONCURRENCY` (8)
- Implementing epic shells
- Spawning a second top-level grok CLI drain (v1: one orchestrator; workers = subagents + worktrees)
- Workers without `model: grok-4.6` (Claude/GPT/Composer inherit)
- Workers running bundled `/implement` until-zero-nits
- Skipping S0, inner review, or runtime proof “to go faster”

---

## Relation to sequential mode

| | Sequential (`seq` or `/solve 1`) | Parallel (default for batches) |
| --- | --- | --- |
| Selection | One at a time | Inventory + waves + refill |
| Guidance | Required for `all` / `N≥2` | Required before workers |
| Parallelism | None | Up to concurrency (default 8) |
| Durability | Local issue branch | Worktree + local issue branch |
| Merge owner | Same session | Orchestrator merge into local `dev` |
| Linear | In Review after local `dev` | In Review after wave merge to local `dev` |
| `main` | `/prb` | `/prb` |
| Failure (`all`) | Continue independents | Cascade-skip deps; continue independents |
