# /solve fast — Orchestrator protocol

Canonical procedure when `FAST_MODE` is true. Sequential `/solve` (no `fast`) ignores this file.

Parent skill: [`../SKILL.md`](../SKILL.md)  
Architecture template: [`architecture-guidance-template.md`](architecture-guidance-template.md)  
Graph schema: [`graph.schema.json`](graph.schema.json)  
Git baseline: [`git-dev-workflow.md`](git-dev-workflow.md)

---

## Purpose

Run multiple eligible Linear leaf issues **in parallel** (worktree-isolated workers), after publishing a shared **batch guidance** package so implementations:

1. Do not diverge or thrash when merging into local **`dev`**
2. Respect **platform / architectural direction** (e.g. Neon over ClickHouse/Convex)
3. **Skip or re-scope** obsolete open tickets
4. Run **migrations/foundations before** features that would be rewritten on an abandoned stack

Canonical analysis procedure: [`batch-guidance.md`](batch-guidance.md).  
Template: [`batch-guidance-template.md`](batch-guidance-template.md).  
The older [`architecture-guidance-template.md`](architecture-guidance-template.md) is a supplement for tech intersections / conflict zones only.

---

## Constants

| Name | Value |
|------|--------|
| `DEFAULT_CONCURRENCY` | `4` when count mode is `all` |
| `MAX_CONCURRENCY` | `8` |
| Integer-mode concurrency | `min(N, MAX_CONCURRENCY)` unless `--concurrency` set |
| `--concurrency M` | Override width; clamp to `1..MAX_CONCURRENCY` |
| Soft worker timeout | ~45–60 minutes; then kill worker, mark issue failed, cleanup |
| Integration branch | local `dev` only |
| Push / PR | never (unless user explicitly asked this session) |

### Count vs concurrency

| Invocation | Issues this run | Max concurrent workers |
|------------|-----------------|-------------------------|
| `/solve fast` | 1 | 1 |
| `/solve 5 fast` | up to **5 issues** (count mode integer) | up to `min(5, 8)` |
| `/solve all fast` | **all** eligible (no soft cap; concurrency 4 ≠ solve 4) | `DEFAULT_CONCURRENCY` (4) |
| `/solve all fast --concurrency 5` | **all** eligible (width 5 ≠ solve 5) | 5 |
| `/solve 12 fast --concurrency 3` | up to 12 | 3 |

**Critical:** default concurrency **4** and default implement effort **5** are **not** issue counts. `/solve all fast` must drain every eligible leaf (refill + end drain gate).

---

## Phase 0 extras (fast)

After normal Phase 0–1 bootstrap:

1. Set `FAST_MODE = true`.
2. Resolve `CONCURRENCY` (table above).
3. Generate `RUN_ID`:

```bash
python3 -c "import uuid; print(uuid.uuid4().hex[:8])"
```

4. Scratch directory (absolute; inline into all paths — do not rely on env vars surviving across shell calls):

```bash
scratch_dir="${TMPDIR:-/tmp}/grok-$(id -u)/solve-fast-${RUN_ID}"
mkdir -p "$scratch_dir/workers" && chmod 700 "${TMPDIR:-/tmp}/grok-$(id -u)" "$scratch_dir"
echo "$scratch_dir"
```

5. Paths:
   - `GUIDANCE_MD` = `$scratch_dir/guidance.md` (authority; also accept legacy name `architecture.md` as alias)
   - `ARCHITECTURE_MD` = same as `GUIDANCE_MD` (legacy alias for worker prompts)
   - `INVENTORY_JSON` = `$scratch_dir/inventory.json`
   - `GRAPH_JSON` = `$scratch_dir/graph.json`
   - `STATE_JSON` = `$scratch_dir/state.json`
   - `WORKER_SUMMARY(id)` = `$scratch_dir/workers/<id>.md`

6. Refresh `main` and local `dev` once (git-dev-workflow) **before** any worker starts. Record `dev_tip = $(git rev-parse dev)`.

7. Trackers: `SOLVED`, `FAILED`, `SKIPPED`, `WORKTREES_CLEANED`, `BRANCHES_DELETED`, `in_progress` map.

---

## F1 — Full eligible inventory

Unlike pure sequential single-issue mode, **pre-scan** the project using **Phase S0 / batch-guidance.md**:

1. Page Linear `list_issues` for team/project (open/actionable states).
2. Apply eligibility (Phase 2B), blocked (2D), epic expansion (2E) → **leaf** set only.
3. Tag platforms, class, supersedes; resolve canonical vs abandoned platforms.
4. Drop **skip**/full-obsolete leaves from the implement set (record in `skipped`).
5. Order remaining by `order_rank` (migrations/foundations first; lowest number is tie-breaker only).
6. Respect count mode: if integer `N`, inventory all eligible for guidance context; implement only until `N` successful merges.
7. Preferred `TEAM-123` (if any): include if eligible and not guidance-skipped; may pin early only if independent of direction deps.
8. Do **not** comment on ordinary scan skips; **do** comment once when canceling/skipping full-obsolete supersedes.

After each wave of merges, **re-scan** Linear for newly unblocked leaves, **patch guidance**, and fold into the graph (append waves). Do not freeze F1 forever.

---

## F2 — Batch guidance package

### Build graph

For each leaf (see `batch-guidance.md` + `graph.schema.json`):

| Field | Source |
|-------|--------|
| `hard_deps` | Linear unfinished `blockedBy` + `depends_on_direction` (migration before re-scoped features) |
| `soft_deps` | Optional related links (informational; do not block start unless promoted) |
| `primary_paths` | Issue body / code map / labels + light `grep` / path inference |
| `platform_tags` / `issue_class` / `action` | S0 tagging |
| `supersedes` / `superseded_by` / `override_scope` | Explicit body + inferred edges |
| `order_rank` / `execution_order` | Migration-first ranking |
| `conflict_zones` | Overlapping `primary_paths` → force serialize listed issues |

**Wave assignment** (execute-plan style):

```text
level(issue) = 0 if hard_deps empty (among unfinished) and not waiting on conflict serialize
level(issue) = max(level(dep) for dep in hard_deps ∪ serialize_predecessors) + 1
```

Within a wave, `merge_order` follows `order_rank`, then lowest issue number.

### Write artifacts

1. Fill [`batch-guidance-template.md`](batch-guidance-template.md) → `GUIDANCE_MD` (also write/copy as `architecture.md` if workers expect that name).
2. Write `INVENTORY_JSON`, `GRAPH_JSON` conforming to [`graph.schema.json`](graph.schema.json).
3. Write `STATE_JSON`:

```json
{
  "run_id": "<RUN_ID>",
  "status": "planning|running|completed|stopped",
  "concurrency": 4,
  "count_mode": "all",
  "effort": 5,
  "guidance_md": "/abs/...",
  "architecture_md": "/abs/...",
  "inventory_json": "/abs/...",
  "graph_json": "/abs/...",
  "canonical_platforms": ["neon", "postgres"],
  "abandoned_platforms": ["clickhouse"],
  "dev_tip_at_start": "<sha>",
  "solved": [],
  "failed": [],
  "skipped": [],
  "worktrees_cleaned": 0,
  "branches_deleted": 0
}
```

### Optional quality boost

Spawn one `plan` or `explore` subagent (read-only) with the issue dump + repo layout to propose tech intersections, shared contracts, conflict edges, and **platform supersession** candidates. **Orchestrator validates and filters** into the package; never trust agent output for eligibility or Linear state.

### Gate

Do **not** start workers until `GUIDANCE_MD` and `GRAPH_JSON` exist, direction confidence is not a blocking **low**, and waves are non-empty (or inventory empty → Phase 9 empty report). Do **not** spawn workers for `status=skipped` / abandoned-platform leaves.

---

## F3 — User report (non-blocking)

```text
Fast plan: K implementable · S skipped · W waves · concurrency C · effort E · run <RUN_ID>
Canonical: neon/postgres · Abandoned: clickhouse
Wave 0: TEAM-123 (migration)
Wave 1: TEAM-80 (rescope after TEAM-123)
Skip: TEAM-67 (abandoned platform)
Guidance: <GUIDANCE_MD>
```

Proceed without waiting for confirmation (unless direction confidence is low or user asked for dry-run only).

---

## F4 — Worker loop

### Claim (orchestrator only)

Before spawn, for each issue about to launch:

1. Assign to me if unassigned.
2. Set **In Progress**.
3. Start comment (note `fast run <RUN_ID>`, wave, architecture path, implement→review, local `dev` only).
4. Parent epic brief comment when expanded.
5. Mark graph status `claimed` → `implementing`.

### Launch worker

`spawn_subagent`:

| Param | Value |
|-------|--------|
| `subagent_type` | `general-purpose` |
| `isolation` | `worktree` |
| `background` | `true` |
| `description` | `[solve-fast] TEAM-123: <title>` |
| capability | full (default general-purpose) |

**Branch prep (orchestrator, main repo, no checkout of issue work on main tree if avoidable):**

- `wave_base_sha` = `git rev-parse dev` at wave start (after prior merges).
- Create branch: prefer Linear `gitBranchName`; else `feat/<team-lower>-<number>-short-slug`.
- `git branch <issue-branch> <wave_base_sha>` (or create from `dev` tip).
- After spawn, push ref into worktree if needed:

```bash
git push <worktree_path> refs/heads/<issue-branch>:refs/heads/<issue-branch>
```

Record `worktree_path`, `worker_task_id`, `base_sha`, `branch` in graph/state.

### Worker prompt (required sections)

```markdown
You are a solve-fast **worker** for a single Linear leaf issue.

## Fast-mode constraints (hard)
- Batch guidance (read fully): <GUIDANCE_MD abs>
- Graph JSON: <GRAPH_JSON abs> — your issue id: <TEAM-N>
- Canonical platforms / abandoned platforms from guidance — **guidance wins** on stack
- Your action: normal|rescope|promote — apply re-scope / override scope from graph
- Obey shared contracts; do not invent parallel APIs/schemas or abandoned stacks
- Worktree only; checkout branch: <issue-branch>
- Base must be wave base / this branch tip; do not merge other issues
- Run full /implement loop (load IMPLEMENT_SKILL_MD) with effort <E>
- Inject custom-implement-instructions.md verbatim
- Verify per AGENTS / issue verification; commit on issue branch:
  <TEAM-N>: short imperative summary
- Write summary to: <WORKER_SUMMARY abs> (include supersession/rescope compliance)
- Do **not** merge to dev/main, push, open PR, or set Linear Done/In Progress
- Do **not** discard unrelated dirty files
- Scope: this leaf only (possibly re-scoped AC)

## Linear issue
- Id, title, url, parent epic, full body, acceptance criteria

## User constraints
<extra>

## Custom solve implement instructions
<verbatim>
```

Worker runs implement→review until 0 open issues, verifies, commits, writes summary, exits.

### Ready-queue

```text
while solved_count < N (or all) and work remains:
  ready = leaves with status pending/claimed, hard_deps all merged,
          conflict predecessors merged, under concurrency budget
  while len(in_progress) < CONCURRENCY and ready non-empty:
    claim + launch next ready (prefer lowest merge_order / issue number)
  wait_any(in_progress task ids)  # long timeout
  for each completed task:
    if success → status ready_to_merge; enqueue merge
    if fail → status failed; cleanup worktree+branch; cascade-skip dependents;
              continue other work (do NOT hard-stop whole run)
  process merge queue in merge_order (F5) before starting issues that depend on merged tips
```

**Launch rule:** never exceed `CONCURRENCY` live worktrees. Prefer **clean-then-launch**: a finished worker should be merged+cleaned (or failed+cleaned) so disk does not accumulate.

### Failure policy (fast ≠ sequential)

| Event | Sequential | Fast |
|-------|------------|------|
| Implement/merge fail | Hard-stop batch | Fail issue; **cascade-skip dependents**; continue independents |
| Worker timeout | n/a | Kill; fail; cleanup; cascade-skip deps |

---

## F5 — Merge into local `dev` (orchestrator only)

Subagent Worktree Protocol (same spirit as `/execute-plan`):

### Success path

```bash
# 1. Capture commits from worktree
git fetch <worktree_path> HEAD --no-tags
commit_sha=$(git -C <worktree_path> rev-parse HEAD)
git cat-file -t "$commit_sha"   # must be commit

# 2. Integrate on main workspace
git checkout dev
# Prefer merge of branch if ref exists in main repo:
git merge <issue-branch> -m "Merge branch '<issue-branch>' into dev"
# Or: git cherry-pick <base_sha>..<commit_sha> --allow-empty

# 3. Verify on dev (repo AGENTS / issue verification)
# 4. Linear Done + completion comment (Phase 8 rules)
# 5. Epic rollup if applicable
# 6. Mandatory cleanup (below)
```

Merge **only** in global `merge_order` among `ready_to_merge` issues. Do not merge a later-order issue first if an earlier ready one is waiting.

On merge conflict: attempt resolve if straightforward; else abort merge, mark issue failed, cleanup, cascade-skip dependents.

### Cleanup contract (mandatory)

**After successful merge + verify + Linear closeout**, immediately:

```text
1. kill_command_or_subagent(worker_task_id) if still alive
2. if worktree_path set and dir exists:
     grok worktree rm --force <worktree_path>
   worktree_cleaned = true
   WORKTREES_CLEANED += 1
3. git branch -d <issue-branch>   # or -D if already on dev and -d refuses
   never delete dev or main
   BRANCHES_DELETED += 1
4. Clear worktree_path in graph; keep commit_sha for the run log
5. status = merged; append to SOLVED
```

**Failed / abandoned** (worktree was created):

```text
1. kill worker if running
2. grok worktree rm --force
3. git branch -D <issue-branch> if exists
4. Linear failure comment; leave In Progress or Blocked
5. cascade-skip dependents still pending
```

**Never** leave worktrees up “for inspection.” Missed cleanup is a bug.

### Live cap

| Moment | Max worktrees |
|--------|----------------|
| Steady state | ≤ `CONCURRENCY` |
| After each successful merge | that issue’s worktree is gone |
| End of run | **0** leftover solve-fast worktrees for this `RUN_ID` |

### End-of-run sweep

```text
for each issue with worktree_path and not worktree_cleaned:
  kill + grok worktree rm --force + delete local issue branch if appropriate
```

Do not remove unrelated user worktrees.

---

## F6 — Refill

After merges (and after wave completions):

1. Re-list Linear for newly unblocked eligible leaves (**all pages**).
2. Update `hard_deps` / waves for new leaves; patch guidance + graph.
3. Continue until:
   - **integer `N`:** `len(SOLVED) >= N`, or
   - **`all`:** no eligible implementable leaves remain **and** drain gate passes, or
   - no progress possible (every remaining open ticket is blocked, other-assignee, failed-this-run with no independent peers left, or guidance-skip).

### Drain gate (`all` only — before Phase 9)

```text
1. Fresh list_issues for team/project (complete pagination)
2. Apply eligibility / blocked / epic expand / guidance skip
3. If any implementable leaf remains → resume F4 ready-queue (do NOT Phase 9)
4. Only when zero implementable leaves → Phase 9 with "Drain verified"
```

**Forbidden in `all`:** stopping after ~5 solves, after the first inventory empties without re-scan, or because concurrency/effort defaults look like a cap.

---

## Phase 9 — Fast summary

**When `count_mode = all`:** emit only after F6 drain gate passes.

```markdown
**Batch (fast):** solved K of target <N|all> · attempted A · failed F · skipped S
**Mode:** /solve <N|all> fast · concurrency C · effort E · run <RUN_ID>
**Team/project:** …
**Architecture:** `<ARCHITECTURE_MD>`
**Waves:** W · max parallelism used: P
**Cleanup:** worktrees removed K · branches deleted K · remaining solve worktrees: 0
**Drain (all only):** verified — no eligible unblocked implementable leaves remain

### Solved
1. [TEAM-123](url) — title · merged → local `dev` · verify pass

### Failed
- [TEAM-125](url) — reason (left In Progress; dependents skipped: …)

### Remaining
- Implementable eligible still open: **none** (required for all after drain gate)
  <!-- integer N may list leftovers -->
**Push/PR:** none (default)
```

Do **not** tell the user to re-run `/solve all fast` to finish work this run should have drained.

---

## Anti-patterns (fast-specific)

- Starting workers before guidance.md + graph.json exist
- Spawning workers for guidance-skipped / abandoned-platform tickets
- Implementing old-stack features before migration deps are **merged** to `dev`
- Worker merges to `dev` or marks Linear Done
- Two workers claiming the same issue
- Starting a dependent before its hard deps are **merged** to `dev`
- Leaving worktrees or issue branches after merge
- Using `git worktree remove` only (use `grok worktree rm --force` for tool-tracked WTs)
- Hard-stopping the entire fast run when one independent issue fails
- Stopping `/solve all fast` after a soft count (~5) or after the first wave without refill + drain gate
- Treating concurrency **4** or effort **5** as a solve-count cap
- Emitting Phase 9 for `all` while implementable leaves remain
- Exceeding `MAX_CONCURRENCY` (8)
- Pushing remotes or opening PRs under default contract
- Implementing epic shells instead of leaves
- Spawning separate `grok` CLI processes (v1 uses subagents + worktrees only)

---

## Relation to sequential mode

| | Sequential | Fast |
|--|------------|------|
| Selection | One at a time, re-select | Inventory + waves + refill |
| Architecture doc | No | Required before workers |
| Parallelism | None | Up to concurrency |
| Merge owner | Same agent after each issue | Orchestrator after worker |
| Failure (`N`) | Hard-stop batch | Cascade-skip deps; continue independents |
| Failure (`all`) | Continue independents after fail | Cascade-skip deps; continue independents |
| `/solve all` exit | Drain gate (re-list Linear) | F6 + drain gate |
| Pre-queue | Allowed after S0 | Required (with refill) |
| Branch cleanup | Optional | **Mandatory** after merge/fail |
