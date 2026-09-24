---
name: solve
description: >
  Use when the user runs /solve, /solve N, /solve all, /solve today, /solve
  <milestone or area>, says "solve the next issue", "pick up the next ticket",
  "solve design related issues", "issues created today", "same dev branch",
  "ignore each other's work", "verify after they are done", Water Cooler
  Protocol, WCP, a /goal to finish today's .WCP/issues ASAP with many
  subagents, or wants open WCP issues completed onto local dev.
argument-hint: "[N|all|today] [seq] [worktree] [--concurrency N] [--effort N] [ISSUE-ID] [milestone|area|today…] [extra constraints…]"
---

# /solve — Next unblocked WCP issue(s) → local `dev`

Select unsolved, unblocked issues from `.WCP/issues/` ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Do not call Linear. The **parent `/solve` session is the orchestrator** (ticket claim and close, git, commit). It **never** implements application source. Each leaf is **one `solve-implementer`** (low reasoning; optional one bug-only `solve-reviewer` on heavy/critical — [`../docs/intensity.md`](../docs/intensity.md)). Verify, then the orchestrator commits onto the long-lived local **`dev`** branch (lowercase) only when `wcp look` shows no live source-file lease. Workers do not commit and do not stash. Default delivery stops there. `/prb` is the deep review.

Multi-issue runs build **batch guidance** first ([`references/batch-guidance.md`](references/batch-guidance.md)) so architectural direction changes (e.g. Neon replacing ClickHouse/Convex) reorder and re-scope work before coding. Then they run **as many independent leaves in parallel as possible on shared local `dev`** ([`references/shared-dev.md`](references/shared-dev.md): **WCP occupancy**, **no worktrees**, orchestrator combined-verifies then commits). Quality gates: S0, WCP ticket status, file leases, runtime proof, drain gate, one assignee per ticket. Worktrees are **opt-in only** (`worktree` / `--worktree` → [`references/fast-mode.md`](references/fast-mode.md)). Occupancy contract: [`../docs/wcp.md`](../docs/wcp.md).

| When | How |
|------|-----|
| `/solve` / `/solve 1` | One worker (no parallelism to use) |
| `/solve N` (`N ≥ 2`), `/solve all`, `/solve today`, scoped drain | **Shared-dev parallel** — [`references/shared-dev.md`](references/shared-dev.md) |
| `worktree` / `--worktree` | Opt-in worktree isolation — [`references/fast-mode.md`](references/fast-mode.md) |
| `seq` / `--sequential` | Escape hatch: one issue at a time |

Do **not** wait for an explicit `fast` flag. `fast` / `--fast` is accepted and ignored (already the batch path).

## Drain contract (`/solve all` — non-negotiable)

When `SOLVE_COUNT_MODE = all` (sequential, worktree, or shared-dev; project-wide **or** scoped, including **created today**):

1. **No soft batch cap.** There is **no** implicit limit of 5 (or any other fixed K). Do not stop because “enough issues were done,” context feels long, a wave finished, or a round number looks complete.
2. **`all` ≠ `5`.** Inner-review is auto-dialed per issue from [`../docs/intensity.md`](../docs/intensity.md) (or `--effort N`, capped at 1). Shared-dev **concurrency** default **all ready** (cap 32). Neither is a solve count. Only a bare integer token sets count mode to `N`.
3. **Continue after every success** until Phase 2 (or F6 / D8 refill) finds **zero** eligible unblocked implementable leaves **in the active set** (`SCOPE` if set, else the project).
4. **Mandatory drain gate before Phase 9:** re-read `.WCP/issues/` per [`references/eligibility.md`](references/eligibility.md). Re-apply **scope filter**, eligibility (2B), blocked (2D), and the guidance skip set. If any implementable leaf remains in the active set → **do not** write the final batch summary; resume the batch loop / ready-queue immediately. A scoped run may stop while the rest of the project still has eligible leaves.
5. **Allowed exits only:**
   - No eligible unblocked implementable leaves remain (true drain), **after** the drain gate passes, or
   - User aborts / stalemate needs human input, or
   - Integer-`N` mode only: `len(SOLVED) >= N`, or
   - Sequential **integer-`N`** only: implement/merge hard-fail stops the batch.
6. **Forbidden exits in `all` mode:** stopping after an arbitrary count (especially ~5); Phase 9 with “re-run `/solve all`” / “re-run `/solve today`” to continue while eligible leaves still exist **in the active set**; treating initial S0 inventory size as a fixed work queue without refill; ending because the first wave is empty without a full re-read of `.WCP/issues/`.
7. **Progress messaging:** interim tallies (`Solved k (all mode) · still draining…`) are fine. The **final** Phase 9 summary runs only when the drain gate passes (or a real allowed exit above).

## Operating contract

- **Batch size** (how many issues this run solves):
  - **Default**: `1` — `/solve` with no count **and no scope** solves a single issue.
  - **Integer `N`**: `/solve N` solves up to **N** eligible issues in parallel (up to `min(N, 32)` workers unless `--concurrency` or `seq`). Inside a scope, `N` is a cap on that cluster.
  - **`all`**: **drain** — keep going until **no** eligible unblocked implementable leaves remain **in the active set** (whole project, or `SCOPE` if set). See [Drain contract](#drain-contract-solve-all--non-negotiable). No soft cap. Mandatory re-read of `.WCP/issues/` before exit. Independent leaves run in parallel.
  - **Scope** (milestone / label / area remainder, or **created today**): resolve per [`references/eligibility.md`](references/eligibility.md) **Scope filter**. A resolved `SCOPE` with no bare `N` sets count mode to **`all` for that cluster only** — drain in parallel among independent in-scope leaves. Do not pick outside `SCOPE`. `/solve today` = created-today window, not a project-wide `/solve all`.
- **Parallelism**: automatic whenever more than one implementable leaf is in play (`N ≥ 2`, `all`, scoped drain, **today**). Default protocol: [Shared-dev](#shared-dev--same-branch-parallel) + [`references/shared-dev.md`](references/shared-dev.md) (same local `dev`, **WCP exclusive file leases**, orchestrator verifies then commits; cap **32**, default = all ready disjoint paths). `worktree` / `--worktree` is the only way onto [Fast mode](#fast-mode--parallel-orchestrator). `seq` / `--sequential` disables parallelism. `fast` / `--fast` is a no-op.
- **Occupancy (WCP)**: every implementer load of `water-cooler-protocol` + [`../docs/wcp.md`](../docs/wcp.md). Orchestrator starts the run and does not assign ids. Workers `wcp name` themselves, then write tests and new files with no claim. They `look` / `acquire --test` / `write-ok` / `release` only for a file that already existed. File overlap waits for the next wave or retargets. A ticket in `blocked/` is a queue status, not a file lease. Never rewind sibling edits. Never push while `WCP_AGENT` is set.
- **Selection (sequential / single issue)**: when batch guidance is active and `seq` was passed, follow **`execution_order`**. When guidance is inactive (`/solve` / `/solve 1` without supersession pressure), lowest issue **number** (e.g. `TW-331` before `TW-343`). Re-validate eligibility after each closeout; refresh guidance when the remaining set changes.
- **Selection (parallel — default for batches)**: full eligible inventory + **shared batch guidance**; launch the **disjoint-path** ready set up to `CONCURRENCY`; refill after combined commits (see shared-dev.md + batch-guidance.md).
- **No epics.** Implement the leaf file. A packaging-only body is skipped in favor of the ids it lists ([`references/eligibility.md`](references/eligibility.md)).
- **Must be unblocked** ([`references/eligibility.md`](references/eligibility.md)).
- **Selection pin**: when `SELECTION_PIN` is set (Identify-approved ids), pick **only** from that ordered set. Do not refill from outside it. Pin ∩ scope when both are set.
- **Queue writes**: claim and close follow [Queue](#queue) and [`../docs/wcp-queue.md`](../docs/wcp-queue.md). Do not edit a ticket you only scanned.
- **Implementation**: **one** `solve-implementer` subagent applying the issue file + [`references/custom-implement-instructions.md`](references/custom-implement-instructions.md). Do **not** run bundled `/implement` until-zero-nits. Optional **one** `solve-reviewer` on heavy/critical (bugs only; nits do not block). The solve orchestrator **does not** write or patch application source itself during Phase 5 (sequential) or shared-dev D5–D6 (resume implementers on verify fail; do not patch).
- **Intensity**: per-issue inner-review yes/no from [`../docs/intensity.md`](../docs/intensity.md) (stamp, else infer; `--effort N` is a hidden inner-review override, capped at 1). Proof is independent of band. Reasoning is **low** via the `solve-implementer` / `solve-reviewer` roles.
- **Custom implement instructions**: always load and inject
  [`references/custom-implement-instructions.md`](references/custom-implement-instructions.md)
  into implementer (and reviewer, when useful) prompts. Edit that file to change solve-time coding policy without forking the whole implement skill. When batch guidance exists, also inject the run’s **`guidance.md`** path (and per-issue supersession/rescope notes) as hard constraints — guidance wins over older tickets on platform/stack.
- **Batch guidance (S0)**: for `/solve all`, `/solve today`, `/solve N` (`N ≥ 2`), and any parallel run, build a guidance package **before** the first implement claim. Procedure: [`references/batch-guidance.md`](references/batch-guidance.md). Template: [`references/batch-guidance-template.md`](references/batch-guidance-template.md). Detects competing architectural directions (e.g. ClickHouse/Convex tickets vs Neon migration), orders migrations first, skips or re-scopes obsolete open tickets, and allows selective override of prior work. Shared-dev still runs S0. File overlap is WCP occupancy ([`../docs/wcp.md`](../docs/wcp.md)). A blocked ticket stays in `blocked/` until its `reason` is satisfied.
- **Integration branch name is always `dev`** — all-lowercase **d-e-v**. Never use capital-`D` `Dev` for checkout, merge, or new work. If a legacy `Dev` ref is found, rename it once to `dev` (see git-dev-workflow) then continue only on `dev`.
- **Git delivery default**:
  1. Refresh from latest `main`
  2. Keep local **`dev`** up to date with `main`
  3. **Parallel (default):** all workers stay on local `dev` (no issue branches, no worktrees). Workers do not commit and do not stash. Orchestrator **combined-verifies after they finish**, then commits on `dev` only when `wcp look` shows no live source-file lease ([`references/shared-dev.md`](references/shared-dev.md)).
  4. **Sequential (`seq` / `/solve 1`):** short-lived issue branch off `dev`. The implementer leaves the tree dirty. The orchestrator verifies, commits only when `wcp look` shows no live source-file lease, then merges into local `dev`.
  5. **Worktree opt-in only:** [`references/fast-mode.md`](references/fast-mode.md) — do not use unless the user passed `worktree`.
- **Do not** `git push`, open a PR, or deploy unless the user explicitly requests it in this session.
- **Closeout**: after the work commit is on local `dev`, the orchestrator sets `files`, `commit`, and `status: done`, then moves the file to `done/` ([`references/multiplayer-linear.md`](references/multiplayer-linear.md)). Workers do not close the ticket. There is no In Review status.
- **Secrets**: never commit `.env`, print Doppler values, tokens, or connection strings.
- **Do not call Linear.** The queue is files.
- **Scope discipline**: satisfy the issue’s acceptance criteria; file follow-ups (e.g. via `/issue`) instead of expanding scope.
- **Dirty tree**: never discard unrelated user changes. Only stage files for this issue. Do not stash another writer's files to make a commit.

## Invocation

```text
/solve
/solve 1
/solve 3
/solve 11
/solve all
/solve today
/solve all --concurrency 6
/solve 12 --concurrency 3
/solve --effort 3
/solve 5 --effort 3
/solve all --effort 2
/solve TW-123
/solve 1 TW-123 also add a unit test for the edge case
/solve --effort 2 TW-123 also add a unit test for the edge case
/solve design related issues
/solve milestone Design
/solve 2 design
/solve all design
/solve 5 seq
```

The `/goal` (or plain) text “finish all Linear issues created today ASAP, many subagents, same local `dev`, ignore each other, verify after” is **`/solve today`**. Do not wait for the user to retype `/solve today`.

### Args

| Arg | Meaning |
|-----|---------|
| `N` (positive integer) | Solve **up to N** eligible issues this run. **Default: 1** when omitted. `N ≥ 2` runs shared-dev parallel with concurrency `min(N, 32)` unless `--concurrency` or `seq`. |
| `all` | **Drain** every eligible unblocked implementable leaf in the active set (project, or `SCOPE`). No soft cap. Shared-dev parallel among ready leaves. Mandatory re-query Linear before Phase 9; only real stop conditions apply (see [Drain contract](#drain-contract-solve-all--non-negotiable)). |
| `today` | **Drain issues created today** (`SCOPE.kind = created`). Shared-dev parallel. Not a project-wide `/solve all`. Same drain gate, in-window only. |
| `seq` / `--sequential` | Force one-at-a-time (escape hatch). Default for batches is shared-dev parallel. |
| `worktree` / `--worktree` | Opt-in worktree isolation ([`references/fast-mode.md`](references/fast-mode.md)). Default parallel does **not** use worktrees. |
| `fast` / `--fast` | Accepted no-op. Batches are already parallel. |
| `--concurrency M` | Max simultaneous workers. Shared-dev: 1–32 (default **all ready**, cap 32; integer `N` → `min(N, 32)`). Worktree opt-in: 1–8. Ignored (warn once) under `seq`. |
| `--effort N` | Hidden inner-review override (0–5), **capped at 1 reviewer**. Default: auto from [`../docs/intensity.md`](../docs/intensity.md). Not Grok reasoning and not bundled `/implement --effort`. |
| `TEAM-123` | Prefer this issue for the **first** slot if unblocked (still validate). If `SCOPE` is set, the id must be in scope. Parallel: include in inventory when eligible. Sequential (`seq`): later slots re-select inside the active set. |
| other text | **Scope first** (created-today before milestone/label/area), then extra implementer constraints. Resolve leftover tokens as **created today**, then a Linear **milestone**, then **label**, then title/label/id **area** ([`references/eligibility.md`](references/eligibility.md) Scope filter). A hit drains that cluster (`all` unless `N` was given). If it is clearly extra constraints and matches no cluster, keep default count **1**. |

### Parsing order

1. Extract `--effort N` / `effort N` (effort flag; do **not** treat this `N` as the solve count).
2. Extract `--concurrency N` / `concurrency N` (worker width; do **not** treat this `N` as the solve count).
3. Detect `seq` / `--sequential` / `sequential` (case-insensitive) → **`FORCE_SEQ = true`**.
4. Detect `fast` / `--fast` (case-insensitive) → ignore (compat no-op).
5. Detect `worktree` / `--worktree` (case-insensitive) → **`WORKTREE_MODE = true`** (opt-in only).
6. Detect **today** from remaining tokens **or the full user message** (case-insensitive):
   - `today` / `created-today` / `created today` / `issues created today` / a `/goal` to finish **today’s** Linear issues → **`TODAY_MODE`**. Phase 1.5 sets `SCOPE.kind = created`.
7. Extract count mode from the remaining tokens:
   - Literal `all` (case-insensitive) → count mode **`all`**.
   - Literal `today` already consumed in step 6; do **not** treat it as a count integer.
   - A bare positive integer token (e.g. `1`, `11`) that is **not** part of `--effort` / `--concurrency` → count mode **`N`**.
   - If neither is present → count mode **unset** (default **`1`** unless Phase 1.5 sets `SCOPE`, including today).
8. Treat a `TEAM-\d+` token as preferred issue id (first issue / inventory pin).
9. Remainder = `REST` (scope query and/or extra constraints). Resolve `SCOPE` in **Phase 1.5** after team/project exist. Then:
   - `SCOPE` set + count unset → **`all`** (this scope).
   - `SCOPE` set + count `N` / `all` → keep that count **inside** the scope.
   - `SCOPE` unset + count unset → **`1`**.
   - `SCOPE` unset: leftover `REST` = extra implementer constraints.
10. Set **`SHARED_DEV`** / **`FAST_MODE`**:
    - `FORCE_SEQ` → both false (sequential loop)
    - `WORKTREE_MODE` and not `FORCE_SEQ` → `FAST_MODE = true`, `SHARED_DEV = false` (opt-in worktrees)
    - else if count is `all` or integer `N ≥ 2` or `TODAY_MODE` (including after Phase 1.5 implicit `all`) → **`SHARED_DEV = true`**, `FAST_MODE = false`
    - else both false (single issue)

**Disambiguation:** `--effort 3` never sets the solve count to 3. `--concurrency 3` never sets the solve count to 3. `/solve 3` means three issues in shared-dev parallel (inner-review per stamp, width up to 3). `/solve all --concurrency 5` drains the active set with **width** 5 (not a cap of 5 issues). `/solve all` means **unlimited until drained** — never treat `--effort` or concurrency **32** as “solve only thirty-two tickets.” `/solve today` means **created today**, shared-dev, drain that window — not project-wide `all`, not a milestone named Today. `/solve design related issues` is a **scoped drain**, not extra constraints and not a project-wide `all`. `fast` does not change behavior. `worktree` is the only way to get worktrees.

### Concurrency defaults

| Invocation | Max concurrent workers |
|------------|-------------------------|
| `/solve` / `/solve 1` | 1 |
| `/solve N` | `min(N, 32)` |
| `/solve all` / `/solve today` / scoped drain | **all ready**, clamp **32** |
| any shared-dev + `--concurrency M` | `clamp(M, 1, 32)` |
| `worktree` opt-in | `min(N, 8)` or **8**; `--concurrency` clamps 1–8 |
| `seq` | 1 |

## Trigger phrases

`/solve`, `/solve N`, `/solve all`, `/solve today`, `/solve design related issues`, `/solve the Design milestone`, `solve the next issue`, `solve the next N issues`, `solve all open issues`, `solve <milestone or area> related issues`, `issues created today`, `tickets created today`, `same dev branch`, `ignore each other's work`, `verify after they are done`, `/goal` to finish today’s Linear issues ASAP, `parallel solve`, `pick up the next ticket`, `work the next unblocked Linear issue`, `next lowest Linear issue`, `complete the next open ticket onto dev`

---

## Phase 0 — Repo bootstrap

1. Confirm workspace root (git repo).
2. Read root `README.md` and applicable `AGENTS.md` / `CLAUDE.md` (and nested AGENTS if working in an app package).
3. Note package manager, verification commands, env tooling (Doppler, etc.).
4. Inspect git status: branch, dirty files, remotes. **Do not** wipe unrelated work.
5. Parse invocation → set **`SOLVE_COUNT_MODE`**, **`FORCE_SEQ`**, **`WORKTREE_MODE`**, **`FAST_MODE`**, **`SHARED_DEV`**, **`CONCURRENCY`**, **`REST`**:
   - `all` if user passed `all`
   - positive integer `N` if user passed bare `N`
   - `today` / created-today / `/goal` today’s issues → `TODAY_MODE` (count **unset** until Phase 1.5 sets scoped `all`)
   - else count **unset** until Phase 1.5
   - `FORCE_SEQ` if `seq` / `--sequential`
   - `WORKTREE_MODE` if `worktree` / `--worktree` (opt-in only)
   - `SHARED_DEV` = not `FORCE_SEQ` and not `WORKTREE_MODE` and (today-mode **or** count is `all` or integer `N ≥ 2`; Phase 1.5 implicit scoped `all` also turns it on)
   - `FAST_MODE` = `WORKTREE_MODE` and not `FORCE_SEQ` only
   - `CONCURRENCY` from `--concurrency` or defaults (see [Concurrency defaults](#concurrency-defaults); shared-dev default **all ready**, cap **32**)
   - Also parse `--effort`, preferred `TEAM-123`, and `REST` (see [Args](#args)). `fast` is ignored.
6. Initialize batch trackers:
   - `SOLVED = []` (list of successfully closed issues this run)
   - `FAILED = []` / `SKIPPED = []` / `SKIPPED_AT_END = []` as needed
   - `ATTEMPTED = 0`
   - `GUIDANCE_REQUIRED` = true when `SOLVE_COUNT_MODE` is `all` (including scoped drain / today), or integer `N ≥ 2`, or `FAST_MODE`, or `SHARED_DEV`, or preferred issue / user constraints force supersession analysis
   - `SCOPE` = unset until Phase 1.5
   - `RUN_ID`, scratch paths when guidance or fast (see batch-guidance.md / fast-mode.md)
   - Fast only: `WORKTREES_CLEANED = 0`, `BRANCHES_DELETED = 0`
7. Resolve absolute paths used later:
   - `SOLVE_SKILL_DIR` = directory containing this `SKILL.md` (from skill load path).
   - `CUSTOM_IMPL_INSTRUCTIONS` = `$SOLVE_SKILL_DIR/references/custom-implement-instructions.md` — **read this file every run** (and again if the file may have changed between batch items).
   - `PROVE_IT_WORKS_MD` = `$SOLVE_SKILL_DIR/../docs/prove-it-works.md` — **read for Phase 6** (and inject the path into implementer/fast-worker prompts).
   - `INTENSITY_MD` = `$SOLVE_SKILL_DIR/../docs/intensity.md` — **read once**; per-issue inner-review auto-dial (Phase 2F / 5C).
   - `GROK_MODELS_MD` = `$SOLVE_SKILL_DIR/../docs/grok-models.md` — **read once**; every spawn in this run uses Grok-only slugs.
   - `ELIGIBILITY_MD` = `$SOLVE_SKILL_DIR/references/eligibility.md` — **read for Phase 2B / 2D / 2E** (shared with `/identify`)
   - `BATCH_GUIDANCE_MD` = `$SOLVE_SKILL_DIR/references/batch-guidance.md` — **read when `GUIDANCE_REQUIRED`**
   - `BATCH_GUIDANCE_TEMPLATE` = `$SOLVE_SKILL_DIR/references/batch-guidance-template.md` — **read when `GUIDANCE_REQUIRED`**
   - `FAST_MODE_MD` = `$SOLVE_SKILL_DIR/references/fast-mode.md` — **read when `FAST_MODE`**
   - `SHARED_DEV_MD` = `$SOLVE_SKILL_DIR/references/shared-dev.md` — **read when `SHARED_DEV`**
   - `WCP_MD` = `$SOLVE_SKILL_DIR/../docs/wcp.md` — **read every run** (occupancy). Player skill: `$SOLVE_SKILL_DIR/../water-cooler-protocol/SKILL.md` (inject path into every implementer prompt).
   - `ARCHITECTURE_TEMPLATE` = `$SOLVE_SKILL_DIR/references/architecture-guidance-template.md` — optional fast supplement; prefer batch guidance as authority
   - `IMPLEMENT_SKILL_MD` = optional, **not used** for the construction loop (standalone `/implement` is a different skill). Do not fail `/solve` if it is missing.
   - Worker spawn types: `solve-implementer` (required), `solve-reviewer` (heavy/critical only). Authority: [`../docs/grok-models.md`](../docs/grok-models.md) + [`../docs/intensity.md`](../docs/intensity.md).
8. **Branch on mode after Phase 1 + 1.5 (+ S0 when required):**
   - Always run Phase 1 (Linear team/project).
   - Always run [Phase 1.5 — Scope](#phase-15--scope-milestone--group--area) when `REST` is non-empty, `TODAY_MODE` is set, or to confirm `SCOPE` is unset.
   - If `GUIDANCE_REQUIRED`: run [Phase S0 — Batch guidance](#phase-s0--batch-guidance-multi-issue) before any claim.
   - If `SHARED_DEV`: follow [Shared-dev](#shared-dev--same-branch-parallel) + [`references/shared-dev.md`](references/shared-dev.md). **Do not** use worktrees. **Do not** run the sequential batch loop or fast-mode.md.
   - Else if `FAST_MODE`: follow [Fast mode](#fast-mode--parallel-orchestrator) + [`references/fast-mode.md`](references/fast-mode.md) (Phases F1–F6), using the S0 package as the architecture/guidance authority. **Do not** run the sequential batch loop below.
   - Else: sequential [Batch loop](#batch-loop--solve-up-to-n-or-all) (Phases 2–8), selecting via guidance `execution_order` when S0 ran.

---

## Phase 1 — Queue

The board is `.WCP/issues/` in this checkout ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Create `open/`, `in-progress/`, `done/`, `canceled/`, and `blocked/` if a run needs a queue and they are missing. Do not create a backlog. Do not call Linear. Reclaim expired ticket leases before selecting.

---

## Phase 1.5 — Scope (today / milestone / group / area)

When `REST` is non-empty **or** `TODAY_MODE` is set, resolve `SCOPE` using
**Scope filter** in [`references/eligibility.md`](references/eligibility.md)
(created-today, then title/body area). Do not call Linear.

- Created today (`/solve today`, “issues created today”, `/goal` today’s
  issues ASAP): `SCOPE.kind = created`, date = session Today’s date. Mention
  the window once. Drain that window (shared-dev unless `seq`). **Not** a
  milestone named Today.
- Unique milestone or label (e.g. `/solve design related issues` → project
  milestone **Design**): set `SCOPE`, mention the match once, then drain that
  cluster unless a bare `N` was given.
- Several matches: ask **once**; do not guess.
- No match + cluster phrasing: stop and ask; do not drain the whole project.
- No match + constraint phrasing: no `SCOPE`; extra constraints only.
- Preferred `TEAM-123` out of scope: skip that pin, say why once, continue in
  scope.

After resolve: if `SCOPE` is set and count is still unset →
`SOLVE_COUNT_MODE = all` and `GUIDANCE_REQUIRED = true`. Unless `FORCE_SEQ`
or `WORKTREE_MODE`: set **`SHARED_DEV = true`**, `FAST_MODE = false`; if
`--concurrency` was not passed, set `CONCURRENCY` to **all ready** (cap 32).
Independent in-scope leaves run in parallel; the drain gate is still in-scope
only (created-today = that calendar date only).

---

## Phase S0 — Batch guidance (multi-issue)

**When:** `GUIDANCE_REQUIRED` (see Phase 0 / 1.5). **Skip** for single-issue solves without supersession pressure. Scoped drains (`SCOPE` + `all`) **do** run S0 on the in-scope leaf set only.

**Full procedure:** [`references/batch-guidance.md`](references/batch-guidance.md).  
**Template:** [`references/batch-guidance-template.md`](references/batch-guidance-template.md).

### S0 summary

```text
1. Inventory eligible leaves (epic expand, blocked filter) + repo architecture truth
2. Tag each leaf: platforms, class (migration|foundation|feature|ops|docs), explicit supersedes
3. Detect direction conflicts (e.g. ClickHouse/Convex tickets vs Neon migration)
4. Resolve canonical vs abandoned platforms (repo truth + newer explicit direction)
5. Classify: normal | promote | rescope | skip/cancel/duplicate
6. Build execution_order: migrations/foundations before features that would be rewritten
7. Write guidance.md + inventory.json + graph.json + state.json under solve-batch scratch dir
8. Report order + skips to user (non-blocking unless direction confidence is low → stop and ask)
```

### Hard ordering example

If open tickets include “build X on ClickHouse” and “migrate to Neon / ClickHouse not launch”:

1. Do **not** implement ClickHouse feature work first just because it has a lower number.
2. Promote Neon migration / foundation tickets early.
3. **Skip** or **re-scope** obsolete open ClickHouse tickets (re-scope = same product outcome on Neon).
4. Inject guidance into every implement prompt so later AC can **selectively override** earlier Done work.

### Done vs open

| Prior ticket state | Superseding open ticket |
|--------------------|-------------------------|
| **Done** | Implement superseder only; override code in scope; do not reopen Done |
| **Open**, fully obsolete | Skip implement; comment + prefer Cancel/Duplicate when high confidence |
| **Open**, outcome still needed on new stack | Re-scope in guidance; implement rewritten AC |

After S0, sequential Phase 2 **selects the next leaf from `execution_order`**, not pure ascending number (still re-validate eligibility each slot). Fast mode uses the same package for waves and skips. Shared-dev uses it for skip/rescope; launch is the disjoint-path ready set (WCP).

---

## Shared-dev — same-branch parallel

**Default for every multi-issue run** (`N ≥ 2`, `all`, `/solve today`, scoped
drain) unless `seq` or `worktree`. When `SHARED_DEV` is true, **stop using
worktrees and the sequential loop**. Follow
**[`references/shared-dev.md`](references/shared-dev.md)** end-to-end.

```text
Phase 0–1  Bootstrap + Linear project (above)
Phase S0   Batch guidance (required) — skip obsolete; occupancy is WCP (disjoint-path waves)
Phase D1   Inventory (created-today when SCOPE.kind is created)
Phase D3   Report + spawn disjoint-path ready workers in one turn (isolation: none, on `dev`)
Phase D5   Wait until every live worker finishes → combined verify
Phase D6   Orchestrator commits on local `dev` only when `wcp look` shows no live source-file lease
Phase D7   Write the hash, mark done, commit the issue file (board still empty)
Phase D8   Refill newly unblocked in-scope leaves; drain gate
Phase 9    Batch summary
```

Non-negotiable: WCP exclusive leases (disjoint-path waves); workers do not
commit; orchestrator does not implement; no worktrees; Linear `blockedBy`
still waits; combined runtime proof after the wave, not per-worker-before-siblings.
Never rewind sibling edits.

If `SHARED_DEV` is false and `FAST_MODE` is true (`worktree` opt-in), use worktree fast mode below.

## Fast mode — parallel orchestrator

**Opt-in only** (`worktree` / `--worktree`). Default parallel is shared-dev above. When `FAST_MODE` is true, **stop using the sequential one-at-a-time batch loop**. Follow **[`references/fast-mode.md`](references/fast-mode.md)** end-to-end. Summary:

```text
Phase 0–1  Bootstrap + Linear project (above)
Phase S0   Batch guidance (required) — guidance.md + graph.json + inventory
Phase F1   Align inventory with S0 (epic expand, blocked filter, skip obsolete)
Phase F2   Ensure package complete: shared contracts, waves, conflict zones, supersession
Phase F3   Report waves/concurrency + direction/skips to user (non-blocking)
Phase F4   CAS-claim → spawn ALL ready workers in one turn (worktrees) →
           implement and verify; workers leave the worktree dirty
Phase F5   Orchestrator merges ready issue branches into local `dev` (merge_order) →
           Linear In Review → cleanup worktrees
Phase F6   Refill newly unblocked leaves; next wave off updated `dev`; drain
Phase 9    Batch summary (remaining worktrees: 0)
```

### Fast rules (non-negotiable)

1. **Guidance first** — no worker starts until `guidance.md` (or equivalent architecture package with supersession/order sections) and `graph.json` exist with shared contracts, tech intersections, waves, conflict zones, **and** platform/supersession decisions.
2. **Workers never merge `dev`/`main`, never open a PR, never set Linear state.** Orchestrator owns CAS claim, merge to local `dev`, and In Review. See [`references/fast-mode.md`](references/fast-mode.md).
3. **Hard deps must be merged to local `dev`** before a dependent worker starts (not merely implemented in another worktree).
4. **Max parallelism** — launch every **ready** leaf up to `CONCURRENCY` (default 8, hard max 8). Live worktrees ≤ `CONCURRENCY`.
5. **Spawn in one turn** — emit every `spawn_subagent` for the current ready set in the **same** orchestrator message (`isolation: "worktree"`, `background: true`). Do **not** wait for worker A before spawning worker B. Then wait with `get_command_or_subagent_output` on the live ids.
6. **Cleanup mandatory** — after each successful merge (and on failure): `kill` worker if needed → `grok worktree rm --force` → delete local issue branch. End of run: **0** leftover solve-fast worktrees for this `RUN_ID`. See cleanup contract in fast-mode.md.
7. **Failure policy** — cascade-skip dependents of a failed issue; **continue** independent issues.
8. **No separate Grok CLI processes** in v1 — use `spawn_subagent` + `isolation: "worktree"`.
9. **Eligibility / epics / Linear comment policy** — same as Phase 2 / Linear issue management (no spam on skips).
10. **`all` drain** — F6 refill + end-of-run Linear re-scan until no eligible leaves remain. Do not Phase 9 while implementable leaves remain.
11. **Quality is not optional** — per-issue intensity inner-review, issue AC, and [`../docs/prove-it-works.md`](../docs/prove-it-works.md) still apply. Parallelism never skips proof or S0.

### Sequential vs parallel

| | Sequential (`seq` or single issue) | Shared-dev (**default** parallel) | Worktree (`worktree` opt-in) |
|--|------------|------|------|
| Selection | One at a time; **guidance order** when S0 ran | Inventory + disjoint-path ready; refill after combined commit | Inventory + waves + refill |
| Batch guidance | **Required** for `all` / `N≥2` | **Required** before workers | **Required** before workers |
| Occupancy | WCP on the issue branch | **WCP** exclusive leases on shared `dev` | Worktrees (no shared WCP board) |
| Parallelism | None | Disjoint-path ready, cap 32 | Up to 8 |
| Isolation | Issue branch | **None** — shared local `dev` | Worktrees |
| Verify | Per issue before merge | **Combined after all live workers finish** | Per issue in worktree, re-verify after merge |
| Merge / commit | Same session after each issue | Orchestrator commits on `dev` (no merge, no stash) | Orchestrator merges wave to local `dev` |
| Failure (`N`) | Hard-stop batch | Continue independents | Cascade-skip deps; continue independents |
| Failure (`all` / today) | Record fail; cascade-skip deps; **continue** independents | Cascade-skip Linear deps only; continue independents | Cascade-skip deps; continue independents |
| Branch/worktree cleanup | Optional | **None** (no worktrees) | **Mandatory** after merge/fail |
| `/solve all` / today exit | Drain gate | D8 + drain gate (today = that date only) | F6 + drain gate |

If `SHARED_DEV` is false and `FAST_MODE` is false, continue with the sequential batch loop below.

---

## Batch loop — solve up to N (or all)

> **Sequential only.** Skip this entire section when `FAST_MODE` or `SHARED_DEV` is true.


After Phase 0–1, run **Phases 2–8 once per issue** until a stop condition:

```text
while true:
  if SOLVE_COUNT_MODE is integer N and len(SOLVED) >= N:
    break   # reached requested count (integer mode only — NEVER apply a fake N in all mode)
  select next issue (Phase 2)   # fresh Linear state every iteration
  if none eligible:
    # tentative empty — in all mode still run drain gate below before Phase 9
    break
  ATTEMPTED += 1
  claim → git hygiene → implement → verify → merge dev → Linear In Review (Phases 3–8)
  if that issue succeeded:
    append to SOLVED
    continue to next issue   # all mode: always continue; N mode: until len(SOLVED) >= N
  else:
    append to FAILED with reason
    if SOLVE_COUNT_MODE is integer N:
      stop the batch   # hard-stop on failure for finite batches
    else:  # all mode
      cascade-skip dependents of the failed issue (guidance graph / blockedBy)
      continue to next independent eligible issue   # do NOT hard-stop the whole drain
```

### Stop conditions

| Condition | Behavior |
|-----------|----------|
| `len(SOLVED) >= N` (**integer mode only**) | Stop; then Phase 9 (no drain gate required beyond count) |
| No eligible issues after Phase 2 | Tentative drain → run [Drain gate](#drain-gate-before-phase-9) in `all` mode |
| Implement hard-fail / unrecoverable verify / dev merge fail | **Integer `N`:** stop batch. **`all`:** leave issue In Progress/Blocked; record FAILED; cascade-skip dependents; **continue** with next independent eligible leaf |
| User aborts / stalemate escalation needs human input | Stop batch; wait for user |
| `all` mode + still eligible after success **or** after an independent failure | **Continue** immediately — never self-limit to 5 or any other soft K |

### Not stop conditions (`all` mode)

Do **not** stop for: “solved five already,” long context, finished first wave, finished S0 inventory without re-scan, wanting a shorter summary, or default effort/concurrency values.

### Multi-issue rules

- **One issue at a time** — finish Phases 2–8 (including merge to local `dev` and Linear **In Review**) before selecting the next.
- **Guidance order when S0 ran** — walk `execution_order` (skip already solved/skipped/failed); re-validate eligibility each slot. Pure lowest-number selection only when S0 did not run.
- **Refresh guidance** after success (and after failures that may unblock others) if Linear gains new eligible leaves or supersession edges; re-rank **remaining** only. In `all` mode, **refill** is required — initial inventory is not a fixed closed set.
- **Preferred `TEAM-123`** applies only to the **first** selection in the batch when it remains eligible and not skipped; subsequent selections follow guidance order (or lowest number if no guidance).
- **Git**: each issue gets its own short-lived branch off the updated `dev` (which already contains prior solves from this batch).
- **Progress**: keep a short running tally visible (e.g. todo or interim note): `Solved k/N` or `Solved k (all mode — still draining)` + skipped/failed counts. Never frame interim progress as final completion while `all` still has eligible leaves.
- **Do not** start issue *k+1* while issue *k* is still open review / unmerged.
- **Do not** implement issues listed as `skip` / full obsolete platform in guidance.

### Drain gate (before Phase 9)

**Required when `SOLVE_COUNT_MODE = all`.** Also recommended after multi-issue `N` if you claim the board is empty.

```text
1. Linear inventory fetch in eligibility.md (team + project + state per
   actionable status; page each state; do not list the project unfiltered)
2. Filter: **scope** (`issue_in_scope` when `SCOPE` is set), eligible states (2B),
   not blocked (2D; out-of-scope blockers stay blocked — do not implement them),
   expand epics (2E), not guidance-skip, not already in SOLVED/FAILED this run
   (unless FAILED leaf is still wrongly open and independents remain —
   independents still count as remaining work)
3. If any implementable leaf remains **in the active set**:
     - Do NOT emit Phase 9 final summary
     - Resume batch loop at Phase 2 / next ready issue
4. Only when zero implementable leaves remain → Phase 9
5. In Phase 9 for all: state explicitly "Drain verified: no eligible unblocked implementable issues remain" (add "in scope \<name\>" when `SCOPE` is set)
```

Then after the loop exits **and** the drain gate passes (for `all`) → **Phase 9** (batch user reply).

---

## Phase 2 — Select the next issue

### 2A. List candidates

Read `.WCP/issues/` per [`references/eligibility.md`](references/eligibility.md). If `SCOPE` is set, keep only `issue_in_scope` before 2B–2E.

### 2B. Eligible states

Canonical: [`references/eligibility.md`](references/eligibility.md). Apply that filter. A `done` file whose `commit` is on local `dev` is finished. Do not call Linear.

### 2C. Sort / pick key

**When `SELECTION_PIN` is set** (Identify or injected): follow
[`references/eligibility.md`](references/eligibility.md) (Selection pin). Walk pin
order; never pick outside it; never refill from outside it.

**When S0 guidance exists** (multi-issue batch, no pin or pin already filtered):

1. Walk `execution_order` from `graph.json` / `guidance.md`.
2. Skip ids already in `SOLVED` or guidance skip/cancel set.
3. For each candidate, re-check eligibility (2B), blocked (2D), epic expansion (2E).
4. Pick the first remaining implementable leaf.
5. If order is empty but eligible issues remain outside the list, **refill S0** (or fall back to lowest number with a warning in the run log).

**When S0 did not run** (single-issue default):

Parse the id: `(\d+)` from frontmatter or the filename. Sort **ascending** among
candidates already filtered by `SCOPE` (if set). Walk in that order; for each,
apply blocker check, then **epic expansion (2E)**, until a **leaf** implementable
issue remains.

If the user named an issue id in the same turn **and this is the first selection of the batch**, start from that issue (still require unblocked; not guidance-skipped; still expand if it is an epic/parent). On later batch slots with guidance, follow `execution_order`; without guidance, use lowest unblocked number.

### 2D. Blocked definition

Canonical: [`references/eligibility.md`](references/eligibility.md) (Blocked).

### 2E. Epic / parent expansion

Canonical: [`references/eligibility.md`](references/eligibility.md) (Epic / parent expansion). `/solve` **does** rollup-Done when all children are terminal (Done/Canceled/Duplicate — **In Review is not terminal**). After a leaf closeout the leaf is In Review, so **do not** rollup the epic in Phase 8.

When starting a child from an epic: note in the **leaf** start comment `via epic TEAM-100`.

### 2F. Inspect before claiming

For the **resolved leaf** issue (after epic expansion), load full issue: description, acceptance criteria, code map, drift check, comments, relations, labels, and **parent epic id/title** when expanded.

**Intensity (required before implement):** follow [`../docs/intensity.md`](../docs/intensity.md). Set `ISSUE_INTENSITY` + `INNER_REVIEW` (`none` | `bugs-only`) for this leaf:

1. If the user passed `--effort N` this run → inner-review per intensity.md (`N` capped at 1); still record the stamp/inferred band.
2. Else parse `## Intensity` / `Band:` (`light`/`standard` → none; `heavy`/`critical` → bugs-only).
3. Else infer from the body + code map (Classify in intensity.md). Ambiguous → bump one band up.
4. Announce once: `Intensity: <band> · inner-review none|bugs-only · <why> · proof on|n/a`.
5. Do not spawn a classifier agent. Do not spawn 2–6 construction reviewers.

**Thin ticket:** if the body fails [`../issue/references/execution-ready-bar.md`](../issue/references/execution-ready-bar.md) (no code map / AC / drift anchors): spawn `/issue` to **upgrade this id** (write the contract back), then implement from the new body. If upgrade cannot reach the bar, skip with reason and take the next number. Do not implement a stub at low.

### 2G. None eligible

End the batch loop (do not invent work). Report in Phase 9:

- Team/project used
- `SCOPE` if set (created today / milestone/label/area) and that the rest of the board was ignored
- How many already solved this run (if any)
- Guidance path (if S0 ran), execution order, and supersession skips
- Candidates considered and why skipped (blocked / in progress other / wrong state / epic with no eligible children / abandoned platform / out of scope / blocked by out-of-scope)

If this is the first selection and nothing is eligible → stop the whole `/solve` with that report. Note when the board is “drained of implementable work” but still has open tickets only because they were **guidance-skipped** as obsolete.

---

## Phase 3 — Claim the ticket

Only for the leaf we are about to implement:

1. Follow [`references/multiplayer-linear.md`](references/multiplayer-linear.md). If every remaining leaf is held by someone else under a live lease, do not take those.
2. Claim: `assignee`, `status: in-progress`, `lease_expires` now + 10 minutes UTC, move to `in-progress/`.
3. Re-read. If `assignee` is not you, abort and pick another.
4. When S0 cancels an obsolete open ticket, cancel that file with `reason` (player skill). Do not cancel a live lease. Do not rewrite tickets you only scanned.

---

## Phase 4 — Git: main + dev hygiene

Follow [references/git-dev-workflow.md](references/git-dev-workflow.md). Summary:

```text
git fetch origin
git checkout main
git merge --ff-only origin/main

# integration branch is always lowercase: dev
git checkout dev 2>/dev/null || git checkout -b dev main
# only if a legacy capital-D branch exists and dev does not:
#   git branch -m Dev dev && git checkout dev
git merge main

git checkout -b <issue-branch>   # from dev; prefer Linear gitBranchName
```

Rules:

- Long-lived integration branch name is **`dev`** only (lowercase `d`). **Never** create, checkout for work, or merge into capital-`D` `Dev`.
- If `dev` has local commits not in main, still **merge main into dev** (don’t reset dev).
- If merge conflicts on `dev`←`main` block progress, resolve them first or stop with a clear report—do not implement the feature on a diverged broken base.
- Never `git push` under default contract.
- Never `git reset --hard` or force-delete user work.
- Start a WCP run on this checkout if none exists ([`../docs/wcp.md`](../docs/wcp.md)). Sequential implementers still lease paths.

---

## Phase 5 — Implement (cheap construction)

**Do not implement the issue yourself.** Spawn **one** `solve-implementer`. Do **not** run bundled `/implement`.

### 5A. Load construction inputs

1. `read_file` on `CUSTOM_IMPL_INSTRUCTIONS`.
2. Confirm spawn types from [`../docs/grok-models.md`](../docs/grok-models.md) (`solve-implementer`, optional `solve-reviewer`).
3. Scratch: `${TMPDIR:-/tmp}/grok-$(id -u)/solve-${RUN_ID:-seq}-${ISSUE}/` (`umask 077`). Summary file: `summary.md`. Inner-review file (if any): `bugs.md`.

### 5B. Build the implementer prompt

Construct a single description string for the implementer:

```markdown
## Linear issue
- Id: <TEAM-123>
- Title: <title>
- URL: <url>
- Parent epic (if expanded): <TEAM-100 — title> or none
- Intensity: <band> · inner-review none|bugs-only · <why> · proof on|n/a
- Description / acceptance criteria:
<full issue body>

## Code map / comments (if any)
<relevant excerpts>

## User constraints for this run
<extra args from /solve invocation, or "none">

## Batch guidance (hard — when S0 ran)
- Path: <abs guidance.md>
- Canonical platforms: …
- Abandoned platforms: …
- This issue: order_rank=… · class=… · action=normal|rescope|promote|…
- Supersedes / superseded by / override scope: …
- Re-scoped AC (if action=rescope): …
- If guidance and the original ticket disagree on platform/stack, **guidance wins**
- Selective override of prior Done/open work is required inside override scope

## Occupancy (WCP) — hard
- Name yourself. Prefer the issue id lowercased (`issue-123`). `wcp name <id> --json`, then export `WCP_AGENT` and `WCP_NAME_TOKEN`. On `name_taken`, pick another id. Do not rename after the token is set.
- Read `$SOLVE_SKILL_DIR/../water-cooler-protocol/SKILL.md` and `$SOLVE_SKILL_DIR/../docs/wcp.md`
- Write the test first, with no claim: `// WCP <id>: <existing-path> <what it proves> (<arch>)`
- New file: write it. No acquire.
- Pre-existing file: look → acquire --test <test-file> → write-ok → re-read disk → edit → release
- Never rewind sibling edits. Never hold a lease through tests. Never push.

## Solve delivery constraints (hard)
- Work only on the current git branch: <issue-branch>
- Scope is this leaf issue only — do not implement the full parent epic
- Do not push, open PRs, merge to dev/main, or update Linear state
- Do not discard unrelated dirty files
- Do not commit. Do not stash. Leave the tree dirty. The solve orchestrator commits only after `wcp look` shows no live source-file lease
- Smallest complete change meeting **current** (possibly re-scoped) acceptance criteria
- Runtime proof: follow `$SOLVE_SKILL_DIR/../docs/prove-it-works.md` (in-scope classes). Matrix green is not enough.

## Custom solve implement instructions
<verbatim contents of custom-implement-instructions.md>
```

### 5C. Inner review

Use `INNER_REVIEW` from Phase 2F (`none` | `bugs-only`). CLI `--effort` already won there (capped at 1). Fast workers use the **per-issue** band unless `--effort` was passed for the run.

### 5D. Spawn the implementer

`spawn_subagent`:

- `subagent_type`: `solve-implementer` (fallback `general-purpose` if the host rejects the type; say so once)
- `model`: `grok-4.6`
- `description`: `[implementer] <ISSUE> <short title>`
- Prompt = 5B text + “Write an implementation summary to `<summary.md>` (paths, decisions, runtime-proof notes). Do not call Linear. Do not push.”

Do **not** pass a fake `effort:` field. Low reasoning comes from the role.

Wait for completion. If it fails (subagent hard-fail, unrecoverable):

- **Integer `N` mode:** **halt the batch** — leave Linear **In Progress**, do not merge to `dev`, do not start the next issue, report in Phase 9.
- **`all` mode:** leave Linear **In Progress** (or **Blocked** if human/external), do not merge to `dev`, append to `FAILED`, cascade-skip dependents, **continue** selecting the next independent eligible leaf.

**Bugs-only inner review** (when `INNER_REVIEW = bugs-only`):

1. Spawn `solve-reviewer`, `model: grok-4.6`, description `[reviewer] bugs only <ISSUE>`.
2. Prompt: read the diff + summary file. Flag **bugs** only (correctness / security / regression this change introduced). Nits go under `## Notes` and must not be `Status: open` bugs. Write `bugs.md`.
3. If `bugs.md` has open bugs: resume the implementer once to fix those bugs only. Re-run the reviewer **once**. Remaining nits do **not** block. Do not loop to zero nits.

**Orchestrator rule:** while construction is active, you **must not** use `write` / `search_replace` / shell to modify application source for the issue. Only the implementer (and the optional reviewer, notes-only) touch product files. You may still run read-only tools, Linear updates that don’t close the issue, and git status/diff inspection.

Long batches: if the parent context is approaching **180k** tokens, `/compact keep the current issue, files in play, failing tests` before the next leaf.

### 5E. After the implementer reports clean

1. Read the summary file (otherwise reconstruct from `git diff` / status).
2. Confirm working tree changes match the issue scope.
3. Delete the cycle scratch dir after Phase 6 commit (do not stage scratch).
4. Proceed to Phase 6 (verify). Do **not** merge to `dev` yet.

---

## Phase 6 — Verify

**Authority:** [`../docs/prove-it-works.md`](../docs/prove-it-works.md). **Read that file.**

Order:

1. Issue **Verification** / acceptance criteria / **Runtime proof** section
2. Repo `AGENTS.md` / README matrix for the touched package(s)
3. **Runtime proof** when in-scope (user-visible, auth, billing, public API, schema, shared helper): drive the real path; visual parity if the ticket names a reference; blast-radius run if shared/schema/auth
4. Typical matrix (adjust per repo): `typecheck`, relevant tests, `build` when AGENTS requires it

Matrix green is **necessary, not sufficient** for in-scope work. “Not performed” is a **fail**.

Fix failures **by resuming the `solve-implementer`** (spawn/resume; not by editing yourself), then re-verify. Do not merge to `dev` or In Review if required checks **or** in-scope runtime proof fail.

### Commit (after verify passes)

The implementer does not commit. The orchestrator does, and only when `wcp look` shows no live source-file lease. If any source-file lease is live, wait. Do not stash that work aside.

On the **issue branch**, stage **only** this issue's paths and commit:

```text
TEAM-123: short imperative summary
```

Use HEREDOC for the commit message. Do not amend. The `commit` field on the issue file is written after this hash exists (Phase 8). That issue-file update is a second commit, and `wcp look` is still empty.

---

## Phase 7 — Merge into local dev

```text
git checkout dev
git merge <issue-branch>    # prefer merge commit or ff; keep history understandable
# confirm dev still includes latest main (merge main again if main moved—rare mid-run)
```

- Leave the short-lived branch locally in **sequential** mode (delete only if merge succeeded and user/repo prefers cleanup—optional).
- **Fast mode:** always delete the local issue branch after successful merge (and remove the worktree) — see [Cleanup contract](references/fast-mode.md#cleanup-contract-mandatory).
- Working tree on **`dev`** at end of successful run when possible.
- **Still no push.**

If merge to dev fails, do not mark Linear Done; fix or report.

---

## Phase 8 — Close the ticket

After the work commit is on local `dev`:

1. Append touched paths to `files`. Write that hash into `commit`.
2. Set `status: done`. Clear `assignee` and `lease_expires`. Move the file to `done/`.
3. If a `blocked/` ticket names this id in `reason`, and this id is now `done`, unblock that ticket to `open/` and leave `reason`.
4. Commit that issue-file update. `wcp look` still shows no live source-file lease. Do not stash.

If verification or the dev merge failed: leave the ticket `in-progress` if you still hold the lease, or `blocked` with `reason` when a human has to answer. Do not set `done` without a hash. In `all` mode the drain continues with other eligible leaves.

---

## Phase 9 — User reply (concise)

### Single issue (`SOLVE_COUNT_MODE = 1` and one attempt)

```markdown
**Solved:** [TEAM-123](url) — <title>
**Via epic:** [TEAM-100](url) — <epic title>   <!-- omit if not expanded -->
**Linear:** leaf In Review (`origin/dev`); epic <left open | rollup Done only if all children terminal>
**Branch:** `<issue-branch>` → merged into local `dev`
**Main:** dev updated from latest `main` before work
**Implement:** intensity <band> · inner-review none|bugs-only · cheap construction
**Verify:** <matrix commands + pass/fail> · **Runtime:** <driven path + observed state | n/a out-of-scope>
**Push/PR:** none (default)
**Notes:** <one line if needed>
```

### Multi-issue batch (`N > 1` or `all`, sequential)

**Only emit this after the [drain gate](#drain-gate-before-phase-9) when mode is `all`.** Never use Remaining to excuse an incomplete `all` drain.

```markdown
**Batch:** solved K of target <N|all> · attempted A · failed F · skipped S
**Mode:** /solve <N|all> · inner-review per issue
**Team/project:** <resolved>
**Scope:** <milestone Name | label X | area "…" | none>
**Guidance:** `<abs path to guidance.md>` · canonical: … · abandoned: …
**Order:** TEAM-… → TEAM-… (migrations/foundations first when relevant)
**Drain (all only):** verified — no eligible unblocked implementable leaves remain [in this scope]

### Solved
1. [TEAM-123](url) — <title> · branch `…` → local `dev` · matrix + runtime proof pass
2. [TEAM-124](url) — …

### Skipped (superseded / abandoned platform)
- [TEAM-67](url) — reason (Canceled/Duplicate/left open per policy)

### Failed (if any)
- [TEAM-125](url) — reason (left In Progress/Blocked; dependents cascade-skipped; other independents continued in all mode)
  <!-- integer N only may say "batch halted" -->

### Remaining
- Implementable eligible still open: **none** (required for all after drain gate)
  <!-- integer N incomplete batch may list remaining and suggest re-run `/solve` / `/solve N` -->
- Open but blocked / other-assignee / guidance-obsolete only: <ids if any>
**Push/PR:** none (default)
```

### Shared-dev batch (`SHARED_DEV`, default parallel including `/solve today`)

**Only emit after D8 + drain gate when mode is `all` / today.** Use the
template in [`references/shared-dev.md`](references/shared-dev.md) Phase 9.
State **Drain verified: no eligible unblocked implementable issues remain**
(add created today \<date\> or in this scope). Do **not** tell the user to
re-run `/solve` / `/solve today` to finish work this run should have continued.
A today-scope run may leave older tickets open — that is not a today-drain failure.

### Fast batch (`FAST_MODE`)

**Only emit after F6 + drain gate when mode is `all`.**

```markdown
**Batch:** solved K of target <N|all> · attempted A · failed F · skipped S
**Mode:** /solve <N|all> · concurrency C · run <RUN_ID>
**Team/project:** <resolved>
**Scope:** <milestone Name | label X | area "…" | none>
**Guidance:** `<abs path to guidance.md>` · canonical: … · abandoned: …
**Waves:** W · max parallelism used: P
**Cleanup:** worktrees removed K · branches deleted K · remaining solve worktrees: 0
**Drain (all only):** verified — no eligible unblocked implementable leaves remain

### Solved
1. [TEAM-123](url) — <title> · merged → local `dev` · matrix + runtime proof pass

### Skipped (superseded / abandoned platform)
- [TEAM-67](url) — reason

### Failed
- [TEAM-125](url) — reason (left In Progress; dependents skipped: …)

### Remaining
- Implementable eligible still open: **none** (required for all after drain gate)
  <!-- integer N may list leftovers and suggest re-run -->
**Push/PR:** none (default)
```

If `all` drained implementable work: state clearly that **Drain verified: no eligible unblocked implementable issues remain** (in this **scope** when `SCOPE` is set; mention leftover out-of-scope board separately). Mention guidance-skipped obsolete, blocked, or other-assignee tickets separately if still open. **Do not** tell the user to re-run `/solve all` to finish work that this run should have continued.

---

## Blocked definition (quick reference)

Canonical table: [`references/eligibility.md`](references/eligibility.md) (Blocked quick reference).

---

## Queue

| Moment | File |
| --- | --- |
| Scanned and skipped | unchanged |
| Phase 3 claim | `in-progress/`, `assignee` = this agent, `lease_expires` = now + 10 minutes |
| While implementing | renew the ticket lease if it would expire; do not hold a source-file lease through tests |
| Phase 8 success | `done/`, `commit` = work hash, lease cleared |
| Failure, still ours | stay `in-progress`, or `blocked` with `reason` when a human must answer |
| Blocker now `done` or `canceled` | dependent moves to `open/`; `reason` stays |

Workers do not close tickets. The orchestrator does. Do not call Linear.

---

## Anti-patterns

- Picking by priority or “interesting” instead of guidance order / lowest unblocked number
- **Implementing obsolete stack work** (e.g. ClickHouse/Convex features) before an open migration that establishes the canonical platform (e.g. Neon)
- Ignoring batch guidance and implementing pure ascending issue numbers across platform conflicts
- Reopening **Done** tickets instead of letting a newer ticket selectively override their code
- Canceling open tickets on **low-confidence** platform inference (escalate instead)
- **Implementing an epic/parent** that still has eligible children instead of expanding to the lowest eligible child
- Marking the **parent epic Done** when open/non-terminal children remain
- Leaving an epic open forever when **all** children are already terminal (must rollup Done)
- Commenting on every skipped candidate during selection (noise)
- Skipping Phase 3 start or Phase 8 completion comments on a claimed leaf
- Posting a Linear comment without `list_comments` first (duplicate closeouts / claims)
- Treating `/solve` as unlimited when the user did not pass `all` **and** did not name a scope (default is **1**)
- Treating `/solve all` as a soft batch of ~5 (or any fixed K) instead of a full drain
- Treating `/solve design related issues` (or other cluster remainder) as implementer constraints or as a **project-wide** drain
- Picking a leaf **outside** `SCOPE` because it is lower-numbered or unblocks the cluster
- Implementing an **out-of-scope** `blockedBy` target in order to continue a scoped drain
- Confusing `--effort N` or `--concurrency N` (or auto-dialed inner-review / concurrency **8**) with solve count `N`
- Running a multi-issue `/solve` one leaf at a time without `seq`
- Spawning ready workers one-after-another (wait for A, then spawn B) instead of one parallel turn
- Running bundled `/implement` until-zero-nits (or spawning 2–6 construction reviewers)
- Passing a fake `effort:` field on `spawn_subagent` instead of `solve-implementer` / `solve-reviewer`
- Implementing a thin stub instead of `/issue`-upgrading the contract
- Ignoring a valid `## Intensity` stamp and re-guessing lighter
- Emitting Phase 9 for `all` while eligible unblocked leaves still exist (“re-run to continue”)
- Skipping the **drain gate** Linear re-query in `all` mode
- Freezing on the initial S0 inventory in `all` mode without refill after merges/closeouts
- Pre-queuing without S0 analysis when multi-issue guidance is required
- Starting the next issue before the current one is merged to local `dev` and closed out (**sequential**; fast: do not start dependents before hard deps are merged to `dev`)
- Continuing a sequential **integer-`N`** batch after an implement/merge failure without user direction
- Hard-stopping sequential **`all`** after one independent failure while other eligible leaves remain
- Implementing the issue **yourself** instead of `solve-implementer`
- Ignoring `custom-implement-instructions.md` or `guidance.md`
- Spawning implementer/reviewer/fast workers without `model: grok-4.6` (or `grok-4.5` only for explore fan-out) — [`../docs/grok-models.md`](../docs/grok-models.md)
- Using capital-`D` **`Dev`** (or any other casing) instead of lowercase **`dev`** for the integration branch
- Creating a new branch named `Dev`
- Pushing `dev` or opening PRs without being asked
- Implementing on `main` directly
- Letting `dev` fall behind `main`
- Marking **Done** from `/solve` (Done is `/prb` after `main`; exception: epic rollup when all children terminal)
- Treating Linear **In Review** as proof the work is on `dev` (skip without the on-dev check)
- Skipping a leaf because its `blockedBy` is In Review but not **Done** / not on `main` — In Review **on local `dev`** unblocks dependents
- Leaving a stale In Review (not on local `dev`) unworked
- Marking In Review without verification or without merge to local `dev`
- Treating typecheck/tests/build as proof for user-visible, auth, billing, API, schema, or shared-helper changes ([`../docs/prove-it-works.md`](../docs/prove-it-works.md))
- Writing “browser smoke not performed” and still In Review
- Starting a second `/solve all` drain on a project that already has foreign live `claimed-by:` comments
- Coding a leaf after a failed claim re-read (CAS miss)
- Committing unrelated dirty files or secrets
- Expanding into a rewrite when the ticket asked for a narrow fix (unless guidance re-scope requires platform rewrite)
- Silently skipping Linear updates on issues we claimed or completed
- **Fast:** starting workers before guidance.md + graph.json exist
- **Fast:** worker merges to `dev`, opens a PR, or sets Linear Done
- **Fast:** leaving worktrees or issue branches after merge/fail (cleanup is mandatory)
- **Fast:** using only `git worktree remove` for tool-created worktrees — use `grok worktree rm --force`
- **Fast:** exceeding concurrency 8 or spawning separate `grok` CLI processes (v1)
- **Fast:** starting a dependent while its hard dep is only done in a worktree, not yet on local `dev`
- Treating `/solve today` as a project-wide `/solve all` or as a milestone named Today
- Using worktrees / issue branches / stash-to-merge for a parallel `/solve` unless the user passed `worktree`
- Verifying (or merging) worker A before spawning or waiting for worker B in shared-dev
- Worker commit or stash on the shared tree
- Orchestrator commit or stash while `wcp look` shows a live source-file lease
- `git add -A` / reset / checkout that discards sibling files on shared `dev`
- Editing a pre-existing file on shared `dev` without WCP look/acquire/write-ok/release
- Launching two workers whose **primary write paths** overlap in the same wave
- Holding a WCP lease through the test runner or combined verify
- `git push` with `WCP_AGENT` set (hooks refuse; `/prb` / `/yeet` unset it)
- UTC-only `createdAt` filter (misses local-morning issues)
- Capping shared-dev at worktree concurrency **8** when more leaves are ready
- Skipping combined runtime proof in shared-dev “to go faster”
- Implementing application source in the parent during shared-dev D5–D6

---

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/issue` | Files thorough tickets only |
| `/implement` | Standalone implement→review until-zero-nits; **not** used by `/solve` |
| `/solve` | Selects Linear issue(s) + **cheap construction** onto **local `dev`**; count via `/solve [N\|all\|today]` (default 1); remainder can **scope** a milestone/label/area or **created-today** drain; batches run **shared-dev** parallel (worktrees only if `worktree`). Deep review is `/prb`. |
| `/start` | Greenfield scaffold + onboard docs + new Linear project; nested `/solve all` with `cwd` = new repo and `SELECTION_PIN` = V1 leaves |
| `/stat` | Read-only full open board, urgent → least; does not implement |
| `/identify` | Human-approved 2–4 leaf batch; upgrades; JIT-claims; nested `/solve` with `SELECTION_PIN` + shared `RUN_ID` (parallel when the pin is overlap-free) |
| `/execute-plan` | Parallel worktree implement from a design-doc PR DAG (not Linear pick/closeout) |
| `/prb` `/yeet` | Ship local `dev`. `/solve` does not push |

---

## Customizing implement behavior

Edit:

```text
~/.grok/skills/solve/references/custom-implement-instructions.md
```

That file is the supported extension point for coding policy under `/solve`. Prefer editing it over forking the entire implement skill. Standalone `/implement` is the user wrapper plus the bundled loop, and it leases on the shared checkout.

Eligibility (2B / 2D / 2E, scope filter, shared with `/identify`) lives in:

```text
~/.grok/skills/solve/references/eligibility.md
```

Batch guidance (multi-issue order, supersession, platform direction) lives in:

```text
~/.grok/skills/solve/references/batch-guidance.md
~/.grok/skills/solve/references/batch-guidance-template.md
~/.grok/skills/solve/references/graph.schema.json
```

Shared-dev orchestration (default parallel: same `dev`, WCP occupancy, orchestrator verifies then commits) lives in:

```text
~/.grok/skills/solve/references/shared-dev.md
~/.grok/skills/docs/wcp.md
```

Occupancy verbs (player):

```text
~/.grok/skills/water-cooler-protocol/SKILL.md
```

Worktree orchestration (opt-in `worktree` only) lives in:

```text
~/.grok/skills/solve/references/fast-mode.md
~/.grok/skills/solve/references/architecture-guidance-template.md
```

`architecture-guidance-template.md` is a **fast supplement**; `guidance.md` from batch guidance is the **authority** for platform direction and supersession.

---

## Git failure

- **Queue missing**: create the status directories only when this run needs a queue. Do not invent tickets. Do not call Linear.
- **Cannot ff main / dev conflicts**: stop with exact commands and conflict list; do not force.
- **`solve-implementer` type rejected**: fall back to `general-purpose` + `model: grok-4.6` once; do not stop.
- **Batch guidance / fast package fails**: stop before claim/workers; report inventory/graph/direction problems; do not spawn parallel implementers or implement obsolete stack work.
- **Fast worktree cleanup fails**: retry `grok worktree rm --force` once; include leftover paths in Phase 9; do not pretend remaining worktrees is 0.
- **Shared-dev combined verify fails**: resume the implementer(s) for the failing leaf paths; do not In Review; do not edit app source in the parent.
- **Push requested later**: user may run a separate ship step; this skill’s default remains local-only.
