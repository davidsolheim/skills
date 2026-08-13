# /solve fast — Orchestrator protocol

Canonical procedure when `FAST_MODE` is true. Sequential `/solve` (no `fast`) ignores this file.

Parent skill: [`../SKILL.md`](../SKILL.md)
Batch guidance: [`batch-guidance.md`](batch-guidance.md)
Claims / Linear states: [`multiplayer-linear.md`](multiplayer-linear.md)
Git baseline: [`git-dev-workflow.md`](git-dev-workflow.md)

---

## Purpose

Run many eligible Linear leaves **in parallel** without losing work or double-claiming.

1. Inventory + batch guidance (IA, supersession, conflict waves).
2. **One** orchestrator per Linear project. Extra CLIs only take unclaimed leaves; they never start a second drain.
3. Workers implement in **worktrees**, then **push an origin issue branch** so a dead CLI does not eat the wave.
4. Orchestrator lands **one PR per wave into `origin/dev`**. Linear **In Review** after that merge.
5. Next wave **rebases on the new `origin/dev`**. `/prb` still owns `main` + prod migrations.

Linear is the claim board (`claimed-by:` CAS). Git branches are backup + merge surface — **never named after an agent**.

---

## Constants

| Name | Value |
| --- | --- |
| `DEFAULT_CONCURRENCY` | `4` when count mode is `all` |
| `MAX_CONCURRENCY` | `8` |
| Integer-mode concurrency | `min(N, MAX_CONCURRENCY)` unless `--concurrency` set |
| `--concurrency M` | Override width; clamp to `1..MAX_CONCURRENCY` |
| Soft worker timeout | ~45–60 minutes; then kill, fail, cleanup, unclaim |
| Issue branch | `solve/<RUN_ID>/<ISSUE>` (e.g. `solve/a1b2c3d4/KEC-780`) |
| Wave branch | `solve/<RUN_ID>/wave-<n>` |
| Integration | **`origin/dev` via wave PR** (not local-only merge) |
| `main` / prod | never under fast-mode. That is `/prb` |

### Count vs concurrency

| Invocation | Issues this run | Max concurrent workers |
| --- | --- | --- |
| `/solve fast` | 1 | 1 |
| `/solve 5 fast` | up to **5 issues** | up to `min(5, 8)` |
| `/solve all fast` | **all** eligible (no soft cap) | `DEFAULT_CONCURRENCY` (4) |
| `/solve all fast --concurrency 5` | **all** eligible | 5 |

Default concurrency **4** and implement effort **5** are **not** issue counts.

---

## Branch rules

- **Do not** name branches after the bot/CLI (`grok-3`, `codex`, `cursor`). Those identities collide (everyone is David on Linear/GitHub).
- Issue branch is always `solve/<RUN_ID>/<ISSUE>` with the Linear identifier (uppercase as Linear shows it).
- Wave branch is `solve/<RUN_ID>/wave-<n>` (`n` starts at 0).
- Never force-push `dev` or `main`. `--force-with-lease` only on a `solve/<RUN_ID>/*` branch this run created, and only to update after rebase onto a moved `origin/dev`.
- If `origin/dev` is missing, create it from `main` once (`git push -u origin dev`) before the first wave.

---

## Phase 0 extras (fast)

After normal Phase 0–1 bootstrap:

1. Set `FAST_MODE = true`.
2. Resolve `CONCURRENCY`.
3. `RUN_ID` = `python3 -c "import uuid; print(uuid.uuid4().hex[:8])"`.
4. Scratch dir:

```bash
scratch_dir="${TMPDIR:-/tmp}/grok-$(id -u)/solve-fast-${RUN_ID}"
mkdir -p "$scratch_dir/workers" && chmod 700 "${TMPDIR:-/tmp}/grok-$(id -u)" "$scratch_dir"
```

5. Paths: `GUIDANCE_MD`, `INVENTORY_JSON`, `GRAPH_JSON`, `STATE_JSON`, `WORKER_SUMMARY(id)` under scratch (same as before).
6. `git fetch origin`. Refresh local `main` from `origin/main`. Checkout/create local `dev`, merge `origin/main` into it, **push `origin/dev` if it did not exist**. Record `dev_tip = $(git rev-parse origin/dev)`.
7. Trackers: `SOLVED`, `FAILED`, `SKIPPED`, `WORKTREES_CLEANED`, `BRANCHES_DELETED`, `WAVE_PRS`, `in_progress`.

If In Progress on this project already has **foreign live `claimed-by:`** comments: **do not drain**. Report and stop, or work around with `/solve N` on unclaimed leaves only.

---

## F1 — Full eligible inventory

Pre-scan via **Phase S0 / batch-guidance.md**:

1. Page Linear `list_issues` for team/project (open/actionable).
2. Eligibility (2B), blocked (2D), epic expand (2E) → **leaves** only.
3. Tag platforms, class, supersedes; skip full-obsolete.
4. Order by `order_rank` (migrations first; lowest number is tie-break).
5. Integer `N`: inventory all for guidance; implement until `N` successful **wave merges to origin/dev**.
6. Do **not** comment ordinary scan skips. Do comment once when canceling a full-obsolete supersede.

After each wave lands on `origin/dev`, **re-scan** Linear, patch guidance, append waves. Do not freeze F1.

---

## F2 — Batch guidance + waves

Same graph fields as before (`hard_deps`, `conflict_zones`, `primary_paths`, supersession).

**Wave assignment:**

```text
level(issue) = 0 if no unfinished hard_deps and not waiting on conflict serialize
level(issue) = max(level(dep) for dep in hard_deps ∪ serialize_predecessors) + 1
```

Within a wave, `merge_order` = `order_rank`, then lowest issue number.

Waves are **heuristics**. Overlapping `primary_paths` serialize. A “non-overlapping” wave can still conflict at PR time — rebase/resolve or fail that leaf; do not force `dev`.

Do **not** start workers until `GUIDANCE_MD` and `GRAPH_JSON` exist and direction confidence is not blocking **low**.

---

## F3 — User report (non-blocking)

```text
Fast plan: K implementable · S skipped · W waves · concurrency C · run <RUN_ID>
Wave 0: TEAM-123 (migration)
Wave 1: TEAM-80, TEAM-91 (independent)
Skip: TEAM-67 (abandoned platform)
Delivery: origin issue branches → one PR per wave into origin/dev → In Review
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
   Then: wave, plan, verify, delivery = origin issue branch then wave PR to `origin/dev` (not Done).
4. **Re-read immediately.** If another run’s claim is newer, abort this leaf.
5. Graph status `claimed` → `implementing`.

Workers **must not** set Linear state.

### Branch + worktree

- `wave_base_sha` = `git rev-parse origin/dev` at wave start (after `git fetch`).
- `issue_branch = solve/<RUN_ID>/<ISSUE>`
- `git branch <issue_branch> <wave_base_sha>` then spawn `isolation: worktree` on that branch.

### Worker prompt (required)

```markdown
You are a solve-fast **worker** for a single Linear leaf.

## Hard constraints
- Read guidance fully: <GUIDANCE_MD>
- Graph JSON: <GRAPH_JSON> — your id: <ISSUE>
- Guidance wins on stack / rescope
- Worktree only; branch: solve/<RUN_ID>/<ISSUE>
- Base is this wave’s origin/dev tip. Do not merge other issues.
- Full /implement loop + custom-implement-instructions.md
- Verify per AGENTS / issue AC; commit on the issue branch:
  <ISSUE>: short imperative summary
- Write summary to <WORKER_SUMMARY>
- **Push** (or leave HEAD ready for orchestrator push):
  git push -u origin HEAD:solve/<RUN_ID>/<ISSUE>
- Do **not** merge origin/dev or main, do **not** open a PR, do **not** set Linear state
- Do **not** discard unrelated dirty files
- Scope: this leaf only
```

If the worker cannot push (auth/network), it must still commit and exit successfully. The **orchestrator MUST push** that worktree HEAD to `origin/solve/<RUN_ID>/<ISSUE>` before treating the leaf as durable. An unpushed successful worker is **not** done.

### Ready-queue

```text
while work remains:
  ready = pending/claimed leaves whose hard_deps and conflict predecessors
          are **merged to origin/dev**, under concurrency budget
  launch until CONCURRENCY
  on worker success → push origin issue branch if missing → status ready_to_merge
  on worker fail → cleanup local WT; comment Linear; leave In Progress/Blocked;
                   cascade-skip dependents; continue independents
  when the current wave’s launched issues are all ready_to_merge or failed → F5 wave PR
```

Never exceed `CONCURRENCY` live worktrees. Prefer clean-then-launch.

---

## F5 — Wave PR into `origin/dev` (orchestrator only)

Do **not** merge issue branches into **local** `dev` as the ship. Local `dev` may track `origin/dev` after the PR merges.

### When

All issues in the current wave are `ready_to_merge` or `failed`, and at least one is `ready_to_merge`. Independent waves never share a PR.

### Build the wave branch

```bash
git fetch origin
git checkout -B solve/<RUN_ID>/wave-<n> origin/dev

# merge_order among ready_to_merge only
for ISSUE in $MERGE_ORDER; do
  git merge --no-ff origin/solve/<RUN_ID>/$ISSUE \
    -m "Merge solve/<RUN_ID>/$ISSUE into wave $n"
  # conflict: abort that merge, mark ISSUE failed, Linear comment, continue others
done

git push -u origin solve/<RUN_ID>/wave-<n>
```

If **every** merge in the wave conflicts, stop the wave, report, do not open an empty PR.

### Open PR (base `dev`)

```bash
gh pr create --base dev --head solve/<RUN_ID>/wave-<n> \
  --title "solve-fast wave <n> (<RUN_ID>)" \
  --body "$(cat <<EOF
Wave <n> of \`/solve all fast\` run <RUN_ID>.

Issues:
- ISSUE — title — origin/solve/<RUN_ID>/ISSUE

Integration target: **origin/dev** (not main).
Prod migrations / main merge: **/prb** after this lands.
EOF
)"
```

Reuse an open PR for the same head if one exists. Record URL in `WAVE_PRS`.

### Merge to `origin/dev` (no user approval)

`origin/dev` does **not** need approval. Merge when CI is green enough to ship to `dev` (failed required checks → do not merge; comment the failing issues).

```bash
gh pr merge <PR> --merge
git fetch origin
git checkout dev
git merge --ff-only origin/dev || git reset --hard origin/dev  # only if local dev has no unique work
```

Never `gh pr merge` into `main` here.

### Linear closeout (after `origin/dev` has the wave)

For each issue that landed in the merged PR:

1. Completion comment: origin issue branch, wave PR URL, `origin/dev` SHA, verify evidence.
2. **In Review** (or stay In Progress if the team has no In Review). **Never Done.**
3. Epic rollup only when all children are terminal.

Issues that failed merge stay In Progress/Blocked with a failure comment. Do not steal foreign claims.

### Preview

If the repo has a `dev`/preview deployment, note the preview URL in the wave closeout when it is knowable. Do not promote production.

### Cleanup (mandatory after wave merge or fail)

```text
1. kill worker if still alive
2. grok worktree rm --force <worktree_path>
3. delete **local** issue branch (never delete origin/dev or main)
4. optional: delete origin issue branch after it is in origin/dev (`git push origin --delete solve/<RUN_ID>/<ISSUE>`)
   Keep origin issue branches if the wave PR did not merge yet (durability)
5. End of run: 0 leftover solve-fast worktrees for this RUN_ID
```

Do not remove unrelated user worktrees.

---

## F6 — Refill

After a wave is on `origin/dev`:

1. `git fetch origin` — next wave base is `origin/dev`.
2. Re-list Linear (all pages). Patch guidance + graph.
3. Continue until integer `N` met, `all` drain gate passes, or nothing eligible remains.

### Drain gate (`all` only)

Fresh `list_issues` + eligibility. If any implementable **unclaimed** leaf remains → resume F4. Do not Phase 9.

---

## Phase 9 — Fast summary

**`all`:** only after drain gate.

```markdown
**Batch (fast):** solved K of <N|all> · failed F · skipped S
**Mode:** /solve <N|all> fast · concurrency C · run <RUN_ID>
**Delivery:** wave PRs → origin/dev
**Wave PRs:** #… (merged) · #… (open)
**origin/dev:** <sha>
**Cleanup:** worktrees 0 remaining
**Drain (all):** verified — no eligible unblocked unclaimed leaves

### Solved (In Review on origin/dev)
1. [TEAM-123](url) — wave PR #N · origin/dev <sha>

### Failed
- [TEAM-125](url) — reason · branch origin/solve/<RUN_ID>/TEAM-125 (if pushed)

**main/prod:** not shipped — run `/prb` when ready
```

---

## Anti-patterns (fast-specific)

- Second `/solve all` / `fast` drain on a project with foreign live claims
- Naming branches after a bot/CLI
- Starting workers before guidance.md + graph.json
- Treating “non-overlapping” as a guarantee (skip rebase when `origin/dev` moved)
- Worker merging `origin/dev` or `main`, opening a main PR, or setting Linear Done
- Leaving a successful worker **unpushed** (work is not durable)
- Merging a dependent wave before hard deps are on **origin/dev**
- Local-only merge to `dev` as the ship (wave PR is the ship)
- `gh pr merge` to **main** from fast-mode
- Marking Linear **Done** from fast-mode
- Hard-stopping the whole run when one independent leaf fails
- Stopping `/solve all fast` after ~5 or after wave 0 without refill + drain gate
- Exceeding `MAX_CONCURRENCY` (8)
- Implementing epic shells
- Spawning a second top-level grok CLI drain (v1: one orchestrator; workers = subagents + worktrees)

---

## Relation to sequential mode

| | Sequential | Fast |
| --- | --- | --- |
| Selection | One at a time | Inventory + waves + refill |
| Guidance | Required for `all` / `N≥2` | Required before workers |
| Parallelism | None | Up to concurrency |
| Durability | Push `origin/dev` after each issue | Push `origin/solve/<run>/<issue>` then wave PR |
| Merge owner | Same session | Orchestrator wave PR into `origin/dev` |
| Linear | In Review after `origin/dev` | In Review after wave PR merges |
| `main` | `/prb` | `/prb` |
| Failure (`all`) | Continue independents | Cascade-skip deps; continue independents |
