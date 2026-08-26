---
name: solve
description: >
  Find the next unsolved, unblocked Linear issue(s) for the current repo/project
  and fully implement each end-to-end using the /implement implement→review→fix
  loop (with solve-specific custom instructions). Epics/parents expand to their
  lowest-numbered eligible child (never implement the epic shell). Count mode:
  /solve or /solve 1 solves one issue (default); /solve N solves up to N issues;
  /solve all drains every eligible unblocked leaf until none remain (no soft cap
  such as 5; mandatory Linear re-check before exit). Multi-issue runs (/solve all,
  /solve N with N≥2, any fast run) first build batch guidance that inventories
  issues, detects platform/supersession conflicts (e.g. older ClickHouse/Convex
  tickets vs newer Neon migration), orders migrations before features that would
  be rewritten, and skips or re-scopes obsolete open tickets. Fast mode solves
  remaining issues in parallel after that guidance (worktree workers, orchestrator
  merges to local dev, mandatory cleanup). Always refresh from latest main, keep
  local dev updated, complete work on a short-lived branch, verify, commit, and
  merge into local `dev`, run build + security/verify checks, then **push to `origin/dev`** automatically. Never merge to `main` or deploy production without explicit user approval (that is `/prb` + approval).
  Auto-resolve Linear team and project from repo context. Use when the user runs
  /solve, /solve N, /solve all, /solve all fast, says "solve the next issue",
  "pick up the next ticket", or wants open issue(s) completed onto `origin/dev`.
argument-hint: "[N|all] [fast] [--concurrency N] [--effort N] [ISSUE-ID] [extra constraints…]"
---

# /solve — Next Unblocked Linear Issue(s) → Local `dev` (via /implement)

Select unsolved, **unblocked** Linear issue(s) for the active repo’s Linear project, implement each with the **full `/implement` loop**, verify, commit, **merge into the long-lived local `dev` branch** (lowercase), run build + security/verify checks, then **`git push origin dev`**. Default delivery stops at **`origin/dev`** for user review. **Never** merge to `main` or production without explicit user approval.

Multi-issue runs build **batch guidance** first ([`references/batch-guidance.md`](references/batch-guidance.md)) so architectural direction changes (e.g. Neon replacing ClickHouse/Convex) reorder and re-scope work before coding.

**Two execution modes:**

| Mode | When | Behavior |
|------|------|----------|
| **Sequential** (default) | No `fast` flag | One issue at a time; guidance order when S0 ran; integer-`N` hard-stops on failure; **`all` continues past independent failures** |
| **Fast** | `fast` or `--fast` | Inventory → batch guidance → parallel workers (worktrees) → orchestrator merge to `dev` → **mandatory cleanup** → refill until drained |

Full fast protocol: [`references/fast-mode.md`](references/fast-mode.md). Batch guidance: [`references/batch-guidance.md`](references/batch-guidance.md).

## Drain contract (`/solve all` — non-negotiable)

When `SOLVE_COUNT_MODE = all` (sequential or fast):

1. **No soft batch cap.** There is **no** implicit limit of 5 (or any other fixed K). Do not stop because “enough issues were done,” context feels long, a wave finished, or a round number looks complete.
2. **`all` ≠ `5`.** Default implement **effort** is 5; default **fast concurrency** is 4. Neither is a solve count. Only a bare integer token sets count mode to `N`.
3. **Continue after every success** until Phase 2 (or F6 refill) finds **zero** eligible unblocked implementable leaves.
4. **Mandatory drain gate before Phase 9:** re-query Linear (`list_issues`, all pages for the team/project). Re-apply eligibility (2B), blocked (2D), epic expand (2E), and guidance skip set. If any implementable leaf remains → **do not** write the final batch summary; resume the batch loop / ready-queue immediately.
5. **Allowed exits only:**
   - No eligible unblocked implementable leaves remain (true drain), **after** the drain gate passes, or
   - User aborts / stalemate needs human input, or
   - Integer-`N` mode only: `len(SOLVED) >= N`, or
   - Sequential **integer-`N`** only: implement/merge hard-fail stops the batch.
6. **Forbidden exits in `all` mode:** stopping after an arbitrary count (especially ~5); Phase 9 with “re-run `/solve all` to continue” while eligible leaves still exist; treating initial S0 inventory size as a fixed work queue without refill; ending because the first wave or first page of Linear results is empty without a full re-scan.
7. **Progress messaging:** interim tallies (`Solved k (all mode) · still draining…`) are fine. The **final** Phase 9 summary runs only when the drain gate passes (or a real allowed exit above).

## Operating contract

- **Batch size** (how many issues this run solves):
  - **Default**: `1` — `/solve` with no count solves a single issue.
  - **Integer `N`**: `/solve N` solves up to **N** eligible issues (sequential: one at a time; fast: up to N with parallelism).
  - **`all`**: **drain the project** — keep going until **no** eligible unblocked implementable leaves remain (see [Drain contract](#drain-contract-solve-all--non-negotiable)). No soft cap. Mandatory Linear re-check before exit.
- **Fast flag**: `fast` or `--fast` enables parallel orchestration (see [Fast mode](#fast-mode--parallel-orchestrator)). Optional `--concurrency M` (1–8) sets max simultaneous workers.
- **Selection (sequential)**: when batch guidance is active (`/solve all`, `/solve N` with `N ≥ 2`, or forced S0), follow **`execution_order`** from the guidance package (migration/platform foundations before features on abandoned stacks; lowest number only as tie-breaker). When guidance is inactive (`/solve` / `/solve 1` without supersession pressure), lowest issue **number** (e.g. `TEAM-331` before `TEAM-343`). Re-validate eligibility after each closeout; refresh guidance when the remaining set changes.
- **Selection (fast)**: full eligible inventory + **shared batch guidance** (platform direction, supersession, waves); refill after merges (see fast-mode.md + batch-guidance.md).
- **Epics / parents**: never implement an epic (or any issue that has child issues) as the work item. When the next candidate is an epic/parent, **expand** into its **lowest-numbered eligible child** and implement that child instead (see [Phase 2E](#2e-epic--parent-expansion)). Recurse if that child is itself a parent. If **all children are completed** (Done/Canceled/Duplicate/Completed), mark the epic **Done** (rollup) and continue — does not consume a solve-count slot.
- **Must be unblocked** (see [Blocked definition](#blocked-definition)).
- **Linear hygiene**: comments and status changes follow [Linear issue management](#linear-issue-management) — comment on issues we **claim or complete**; do **not** comment on every skipped candidate.
- **Implementation**: **always** via the bundled **`/implement`** skill procedure (implementer + reviewer subagents, loop until 0 open review issues). The solve orchestrator **does not** write or patch application source itself during Phase 5 (sequential) or during worker implement loops (fast — workers host implement; orchestrator still does not edit app source except merge conflict resolution on `dev`).
- **Custom implement instructions**: always load and inject
  [`references/custom-implement-instructions.md`](references/custom-implement-instructions.md)
  into implementer (and reviewer, when useful) prompts. Edit that file to change solve-time coding policy without forking the whole implement skill. When batch guidance exists, also inject the run’s **`guidance.md`** path (and per-issue supersession/rescope notes) as hard constraints — guidance wins over older tickets on platform/stack.
- **Batch guidance (S0)**: for `/solve all`, `/solve N` (`N ≥ 2`), and any fast run, build a guidance package **before** the first implement claim. Procedure: [`references/batch-guidance.md`](references/batch-guidance.md). Template: [`references/batch-guidance-template.md`](references/batch-guidance-template.md). Detects competing architectural directions (e.g. ClickHouse/Convex tickets vs Neon migration), orders migrations first, skips or re-scopes obsolete open tickets, and allows selective override of prior work.
- **Integration branch name is always `dev`** — all-lowercase **d-e-v**. Never use capital-`D` `Dev` for checkout, merge, or new work. If a legacy `Dev` ref is found, rename it once to `dev` (see git-dev-workflow) then continue only on `dev`.
- **Git delivery default** (per issue):
  1. Refresh from latest `main`
  2. Keep local **`dev`** up to date with `main`
  3. Work on a short-lived issue branch off `dev`
  4. Run implement loop + verify + commit on that branch
  5. **Merge into local `dev`**
  6. Run **build + security/verify** (no push until green)
  7. **`git push origin dev`** (automatic; no approval)
  8. **Fast only:** immediately remove the worker worktree and delete the local issue branch (see cleanup contract in fast-mode.md)
- **Long-lived `dev` (hard):** every project must have a long-lived **`dev`** branch **locally and on `origin`**. Create `origin/dev` from `main` if missing (`git push -u origin dev`). Never use capital-`D` `Dev`.
- **Delivery = `origin/dev` (automatic):** after merge to local `dev`, run the repo’s **build + security/verify** checks; only then **`git push origin dev`**. No user approval needed for `origin/dev`.
- **Main/prod still gated:** never merge to `main`, never deploy production, never `gh pr merge` under `/solve`. Promoting `dev` → `main`/prod is **`/prb`** and requires **explicit user approval** (quiet CI is never prod authority).
- **Linear closeout**: after merge to local `dev`, verification, and push to `origin/dev` → **In Review** (or stay In Progress if the team has no In Review) with evidence. **Do not mark Done.** `/prb` marks Done after merge to `main`. Full claim protocol: [`references/multiplayer-linear.md`](references/multiplayer-linear.md). In fast mode the **orchestrator** owns claim + In Review (workers must not set Linear state).
- **Secrets**: never commit `.env`, print Doppler values, tokens, or connection strings.
- **Linear MCP**: `search_tool` then `use_tool`. Read schemas before calling. Literal newlines in markdown bodies.
- **Scope discipline**: satisfy the issue’s acceptance criteria; file follow-ups (e.g. via `/issue`) instead of expanding scope.
- **Dirty tree**: never discard unrelated user changes. Only stage files for this issue.

## Invocation

```text
/solve
/solve 1
/solve 3
/solve 11
/solve all
/solve fast
/solve 5 fast
/solve all fast
/solve all fast --concurrency 6
/solve 12 fast --concurrency 3
/solve all --fast --concurrency 4
/solve --effort 3
/solve 5 --effort 3
/solve 5 fast --effort 3
/solve all --effort 2
/solve TEAM-123
/solve 1 TEAM-123 also add a unit test for the edge case
/solve --effort 2 TEAM-123 also add a unit test for the edge case
```

### Args

| Arg | Meaning |
|-----|---------|
| `N` (positive integer) | Solve **up to N** eligible issues this run. **Default: 1** when omitted. Sequential: one at a time. Fast: up to N with concurrency `min(N, 8)` unless `--concurrency` set. |
| `all` | **Drain** every eligible unblocked implementable leaf. No soft cap. Mandatory re-query Linear before Phase 9; only real stop conditions apply (see [Drain contract](#drain-contract-solve-all--non-negotiable)). |
| `fast` / `--fast` | Enable **fast mode**: batch guidance + parallel worktree workers + orchestrator merge/cleanup. Full protocol in [`references/fast-mode.md`](references/fast-mode.md). |
| `--concurrency M` | Fast only. Max simultaneous workers (1–8). Default: `min(N, 8)` in integer mode; **4** in `all fast` mode. Ignored (warn once) if not in fast mode. |
| `--effort N` | Implement effort 1–5 (reviewer count / specialists). Default: value in `custom-implement-instructions.md` (currently **5** — high/max rigor), else **5**. |
| `ISSUE-ID` (e.g. `TEAM-123`, `ENG-12`) | Prefer this Linear issue for the **first** slot if unblocked (still validate). Sequential: later slots re-select lowest. Fast: include in inventory when eligible. Examples use `TEAM-N`; real teams use their own prefix. |
| other text | Extra constraints for the implementer prompt (applied to each issue in the batch unless clearly first-issue-only). |

### Linear issue identifiers (operational)

Linear issue ids are **`PREFIX-NUMBER`** (e.g. `ENG-12`, `OPS-9`, `TEAM-123`). Operational parse/sort is **prefix-agnostic**:

| Use | Rule |
|-----|------|
| Match preferred id token | `[A-Za-z][A-Za-z0-9]*-\d+` (case-insensitive prefix; trailing digits are the number) |
| Sort / “lowest number” | Parse **trailing digits** from the identifier (`PREFIX-(\d+)` → integer). Do **not** require the literal prefix `TEAM`. |
| Branch names | Prefer Linear’s `gitBranchName` / identifier fields when present; else `feat/<prefix-lower>-<number>-short-slug` |

`TEAM-123` / `TEAM-100` appear in examples and templates only as stand-ins for any real team prefix.

### Parsing order

1. Extract `--effort N` / `effort N` (effort flag; do **not** treat this `N` as the solve count).
2. Extract `--concurrency N` / `concurrency N` (fast width; do **not** treat this `N` as the solve count).
3. Detect `fast` or `--fast` (case-insensitive) → **`FAST_MODE = true`**.
4. Extract count mode from the remaining tokens:
   - Literal `all` (case-insensitive) → count mode **`all`**.
   - A bare positive integer token (e.g. `1`, `11`) that is **not** part of `--effort` / `--concurrency` → count mode **`N`**.
   - If neither is present → count mode **`1`** (default).
5. Treat a Linear issue-id token matching `[A-Za-z][A-Za-z0-9]*-\d+` (e.g. `TEAM-123`, `ENG-12`) as preferred issue id (first issue / inventory pin).
6. Remainder = user constraints.

**Disambiguation:** `--effort 3` never sets the solve count to 3. `--concurrency 3` never sets the solve count to 3. `/solve 3` means three issues at default effort. `/solve 3 fast` means up to three issues in fast mode with concurrency up to 3. `/solve all fast --concurrency 5` drains the project with **width** 5 (not a cap of 5 issues). `/solve all` / `/solve all fast` means **unlimited until drained** — never treat default effort **5** or concurrency **4** as “solve only five (or four) tickets.”

### Concurrency defaults (fast only)

| Invocation | Max concurrent workers |
|------------|-------------------------|
| `/solve fast` | 1 |
| `/solve N fast` | `min(N, 8)` |
| `/solve all fast` | **4** |
| any fast + `--concurrency M` | `clamp(M, 1, 8)` |

## Trigger phrases

`/solve`, `/solve N`, `/solve all`, `/solve all fast`, `/solve N fast`, `solve the next issue`, `solve the next N issues`, `solve all open issues`, `solve all fast`, `parallel solve`, `pick up the next ticket`, `work the next unblocked Linear issue`, `next lowest Linear issue`, `complete the next open ticket onto dev`

---

## Phase 0 — Repo bootstrap

1. Confirm workspace root (git repo).
2. Read root `README.md` and applicable `AGENTS.md` / `CLAUDE.md` (and nested AGENTS if working in an app package).
3. Note package manager, verification commands, env tooling (Doppler, etc.).
4. Inspect git status: branch, dirty files, remotes. **Do not** wipe unrelated work.
5. Parse invocation → set **`SOLVE_COUNT_MODE`**, **`FAST_MODE`**, **`CONCURRENCY`**:
   - `all` if user passed `all`
   - positive integer `N` if user passed bare `N`
   - else **`1`** (default)
   - `FAST_MODE` if `fast` / `--fast`
   - `CONCURRENCY` from `--concurrency` or defaults (see [Concurrency defaults](#concurrency-defaults-fast-only))
   - Also parse `--effort`, preferred issue id (`PREFIX-N`), and extra constraints (see [Args](#args)).
6. Initialize batch trackers:
   - `SOLVED = []` (list of successfully closed issues this run)
   - `FAILED = []` / `SKIPPED = []` / `SKIPPED_AT_END = []` as needed
   - `ATTEMPTED = 0`
   - `GUIDANCE_REQUIRED` = true when `SOLVE_COUNT_MODE` is `all`, or integer `N ≥ 2`, or `FAST_MODE`, or preferred issue / user constraints force supersession analysis
   - `RUN_ID`, scratch paths when guidance or fast (see batch-guidance.md / fast-mode.md)
   - Fast only: `WORKTREES_CLEANED = 0`, `BRANCHES_DELETED = 0`
7. Resolve absolute paths used later:
   - `SOLVE_SKILL_DIR` = directory containing this `SKILL.md` (from skill load path).
   - `CUSTOM_IMPL_INSTRUCTIONS` = `$SOLVE_SKILL_DIR/references/custom-implement-instructions.md` — **read this file every run** (and again if the file may have changed between batch items).
   - `PROVE_IT_WORKS_MD` = `$SOLVE_SKILL_DIR/../docs/prove-it-works.md` — **read for Phase 6** (and inject the path into implementer/fast-worker prompts).
   - `BATCH_GUIDANCE_MD` = `$SOLVE_SKILL_DIR/references/batch-guidance.md` — **read when `GUIDANCE_REQUIRED`**
   - `BATCH_GUIDANCE_TEMPLATE` = `$SOLVE_SKILL_DIR/references/batch-guidance-template.md` — **read when `GUIDANCE_REQUIRED`**
   - `FAST_MODE_MD` = `$SOLVE_SKILL_DIR/references/fast-mode.md` — **read when `FAST_MODE`**
   - `ARCHITECTURE_TEMPLATE` = `$SOLVE_SKILL_DIR/references/architecture-guidance-template.md` — optional fast supplement; prefer batch guidance as authority
   - `IMPLEMENT_SKILL_MD` = resolve the implement skill:
     1. `$HOME/.grok/skills/implement/SKILL.md` if present
     2. else `$HOME/.grok/bundled/skills/implement/SKILL.md`
     3. else report error and stop (cannot run implement loop)
   - `IMPLEMENT_SKILL_DIR` = dirname of that file (personas: `../shared/personas/` relative to bundled layout; memory helper: `$IMPLEMENT_SKILL_DIR/scripts/memory.py`)
8. **Branch on mode after Phase 1 (+ S0 when required):**
   - Always run Phase 1 (Linear team/project).
   - If `GUIDANCE_REQUIRED`: run [Phase S0 — Batch guidance](#phase-s0--batch-guidance-multi-issue) before any claim.
   - If `FAST_MODE`: follow [Fast mode](#fast-mode--parallel-orchestrator) + [`references/fast-mode.md`](references/fast-mode.md) (Phases F1–F6), using the S0 package as the architecture/guidance authority. **Do not** run the sequential batch loop below.
   - Else: sequential [Batch loop](#batch-loop--solve-up-to-n-or-all) (Phases 2–8), selecting via guidance `execution_order` when S0 ran.

---

## Phase 1 — Resolve Linear team & project (automatic)

Same priority order as `/issue`:

1. Repo docs: `AGENTS.md`, `README.md`, `.linear-project`, `.linear.json` / yaml
2. Memory / recent commits / branch names with issue prefixes (`PREFIX-N`, e.g. `TEAM-123`, `ENG-12`)
3. Linear: `list_teams` → `list_projects` for that team; prefer active project matching product/repo name from docs, skip archived/completed umbrellas

If unresolved, ask **once** with top candidates. Reuse resolution for the rest of the session **and for every issue in this batch**.

---

## Phase S0 — Batch guidance (multi-issue)

**When:** `GUIDANCE_REQUIRED` (see Phase 0). **Skip** for single-issue solves without supersession pressure.

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

After S0, sequential Phase 2 **selects the next leaf from `execution_order`**, not pure ascending number (still re-validate eligibility each slot). Fast mode uses the same package for waves and skips.

---

## Fast mode — parallel orchestrator

When `FAST_MODE` is true, **stop using the sequential one-at-a-time batch loop**. Instead follow **[`references/fast-mode.md`](references/fast-mode.md)** end-to-end. Summary:

```text
Phase 0–1  Bootstrap + Linear project (above)
Phase S0   Batch guidance (required for fast) — guidance.md + graph.json + inventory
Phase F1   Align inventory with S0 (epic expand, blocked filter, skip obsolete)
Phase F2   Ensure package complete: shared contracts, waves, conflict zones, supersession
Phase F3   Report waves/concurrency + direction/skips to user (non-blocking)
Phase F4   CAS-claim → worktree workers → implement/verify/commit →
           push origin/solve/<RUN_ID>/<ISSUE> (durable; never agent-named)
Phase F5   Orchestrator: one PR per wave (head solve/<RUN_ID>/wave-N, base origin/dev) →
           merge to origin/dev (no main) → Linear In Review → cleanup worktrees
Phase F6   Fetch origin/dev; refill newly unblocked leaves; next wave rebases; drain
Phase 9    Fast batch summary (wave PRs, origin/dev SHA, remaining worktrees: 0)
```

### Fast rules (non-negotiable)

1. **Guidance first** — no worker starts until `guidance.md` (or equivalent architecture package with supersession/order sections) and `graph.json` exist with shared contracts, tech intersections, waves, conflict zones, **and** platform/supersession decisions.
2. **Workers never merge `origin/dev`/`main`, never open a main PR, never set Linear state.** They **do** push `origin/solve/<RUN_ID>/<ISSUE>`. Orchestrator owns CAS claim, wave PR into `origin/dev`, and In Review. See [`references/fast-mode.md`](references/fast-mode.md).
3. **Hard deps must be merged to `origin/dev`** (wave PR merged) before a dependent worker starts (not merely implemented in another worktree).
4. **Concurrency cap** — live worktrees ≤ `CONCURRENCY` (max 8).
5. **Cleanup mandatory** — after each successful merge (and on failure): `kill` worker if needed → `grok worktree rm --force` → delete local issue branch. End of run: **0** leftover solve-fast worktrees for this `RUN_ID`. See cleanup contract in fast-mode.md.
6. **Failure policy** — cascade-skip dependents of a failed issue; **continue** independent issues.
7. **No separate Grok CLI processes** in v1 — use `spawn_subagent` + `isolation: "worktree"`.
8. **Eligibility / epics / Linear comment policy** — same as Phase 2 / Linear issue management (no spam on skips).
9. **`all` drain** — F6 refill + end-of-run Linear re-scan until no eligible leaves remain (same drain contract as sequential `all`). Do not Phase 9 while implementable leaves remain.

### Sequential vs fast

| | Sequential | Fast |
|--|------------|------|
| Selection | One at a time; **guidance order** when S0 ran | Inventory + waves + refill |
| Batch guidance | **Required** for `all` / `N≥2` | **Required** before workers |
| Parallelism | None | Up to concurrency |
| Merge owner | Same session after each issue | Orchestrator wave PR → origin/dev |
| Failure (`N`) | Hard-stop batch | Cascade-skip deps; continue independents |
| Failure (`all`) | Record fail; cascade-skip deps; **continue** independents | Cascade-skip deps; continue independents |
| Pre-queue | Allowed **after** S0 (`execution_order`); re-validate each slot | Required (with refill) |
| Branch/worktree cleanup | Optional | **Mandatory** after merge/fail |
| `/solve all` exit | Only after **drain gate** (re-list Linear → zero eligible) | Same: F6 + drain gate before Phase 9 |

If `FAST_MODE` is false, continue with the sequential batch loop below.

---

## Batch loop — solve up to N (or all)

> **Sequential only.** Skip this entire section when `FAST_MODE` is true.


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
- **Preferred issue id** (`PREFIX-N`, e.g. `TEAM-123`) applies only to the **first** selection in the batch when it remains eligible and not skipped; subsequent selections follow guidance order (or lowest number if no guidance).
- **Git**: each issue gets its own short-lived branch off the updated `dev` (which already contains prior solves from this batch).
- **Progress**: keep a short running tally visible (e.g. todo or interim note): `Solved k/N` or `Solved k (all mode — still draining)` + skipped/failed counts. Never frame interim progress as final completion while `all` still has eligible leaves.
- **Do not** start issue *k+1* while issue *k* is still open review / unmerged.
- **Do not** implement issues listed as `skip` / full obsolete platform in guidance.

### Drain gate (before Phase 9)

**Required when `SOLVE_COUNT_MODE = all`.** Also recommended after multi-issue `N` if you claim the board is empty.

```text
1. list_issues for team/project — page until complete (do not trust first page only)
2. Filter: eligible states (2B), not blocked (2D), expand epics (2E), not guidance-skip,
   not already in SOLVED/FAILED this run (unless FAILED leaf is still wrongly open and
   independents remain — independents still count as remaining work)
3. If any implementable leaf remains:
     - Do NOT emit Phase 9 final summary
     - Resume batch loop at Phase 2 / next ready issue
4. Only when zero implementable leaves remain → Phase 9
5. In Phase 9 for all: state explicitly "Drain verified: no eligible unblocked implementable issues remain"
```

Then after the loop exits **and** the drain gate passes (for `all`) → **Phase 9** (batch user reply).

---

## Phase 2 — Select the next issue

### 2A. List candidates

Using Linear MCP (`list_issues`):

- Filter by **team** and **project** when known
- Include open/actionable states; exclude archived if the tool allows
- Fetch enough pages to find the true lowest number (do not stop at first page if numbers might be older)

### 2B. Eligible states

Include:

- Backlog, Todo, Ready, Triage, Unstarted, and team equivalents
- **In Progress** only if **unclaimed** (no live foreign `claimed-by:` comment < 60 min) **or** already claimed by **this run**. See [`references/multiplayer-linear.md`](references/multiplayer-linear.md)

Exclude:

- Done, Canceled, Cancelled, Duplicate, Completed
- **In Review** (already on `origin/dev`; waiting on `/prb` to merge `main`. Not implementable. In Review is **not** terminal.)
- In Progress **claimed by another run** (foreign live `claimed-by:` comment)
- In Progress assigned to **someone else** (when assignee is not this Linear user)
- Issues whose only remaining work is explicitly “wait for external X” with X unavailable

### 2C. Sort / pick key

**When S0 guidance exists** (multi-issue batch):

1. Walk `execution_order` from `graph.json` / `guidance.md`.
2. Skip ids already in `SOLVED` or guidance skip/cancel set.
3. For each candidate, re-check eligibility (2B), blocked (2D), epic expansion (2E).
4. Pick the first remaining implementable leaf.
5. If order is empty but eligible issues remain outside the list, **refill S0** (or fall back to lowest number with a warning in the run log).

**When S0 did not run** (single-issue default):

Parse the identifier number from any Linear id: `PREFIX-(\d+)` → integer (regex: `[A-Za-z][A-Za-z0-9]*-(\d+)`). Sort **ascending by that number**. Walk candidates in that order; for each, apply blocker check, then **epic expansion (2E)**, until a **leaf** implementable issue remains. Do **not** require a literal `TEAM-` prefix.

If the user named an issue id in the same turn **and this is the first selection of the batch**, start from that issue (still require unblocked; not guidance-skipped; still expand if it is an epic/parent). On later batch slots with guidance, follow `execution_order`; without guidance, use lowest unblocked number.

### 2D. Blocked definition

Treat as **blocked** (skip) if any of:

- Open `blockedBy` relations to unfinished issues
- State/label clearly means Blocked
- Description/comments require missing credentials, human product decision, or unpaid external dependency you cannot resolve in-repo
- Duplicate of an open issue (work the original instead if lower number / canonical)
- Guidance marks the leaf `skip` / full obsolete platform (do not implement as written; record in `SKIPPED`)

If blocked, record why briefly and continue to the next candidate in order.

### 2E. Epic / parent expansion

**Never implement a parent/epic as the coding target** when it has children. Expand into a child instead.

#### Detect parent / epic

Treat the candidate as a **parent (epic)** if **any** of:

1. `list_issues` with `parentId` set to this issue’s id/identifier returns **one or more** children (canonical check — use Linear MCP; page if needed).
2. Issue is labeled **Epic** (or team equivalent) **and** has children under (1).
3. Issue type/name clearly indicates Epic **and** has children under (1).

If it looks like an “Epic” label/type but has **zero** children → **not** expandable; treat as a normal leaf ticket (or skip as too thin per 2F if it is only a placeholder).

#### Expand procedure

```text
function resolve_work_issue(candidate):
  if candidate is blocked → return skip (no Linear comment), try next global candidate
  children = list_issues(parentId=candidate)   # all pages
  if children is empty:
    return candidate   # leaf — implement this
  # parent/epic: do NOT implement candidate as code work
  eligible_children = children that pass 2B eligible states and 2D unblocked
  sort eligible_children by trailing issue number (PREFIX-(\d+) → int) ascending
  if eligible_children is empty:
    if every child is terminal (Done / Canceled / Cancelled / Duplicate / Completed):
      # epic rollup — free housekeeping, does not count toward SOLVE_COUNT_MODE
      comment on epic: all children complete → marking epic Done
      set epic state Done
      return skip (continue to next global candidate)
    else:
      # e.g. children blocked / in progress elsewhere / canceled mix with open blocked
      record "epic <id> has no eligible children (not all terminal)" → skip, no comment
      try next global candidate
  # pick lowest-numbered eligible child; recurse (nested parents)
  return resolve_work_issue(eligible_children[0])
```

Rules:

- **One expansion per selection** yields a single leaf issue for this batch slot.
- Prefer the **lowest-numbered eligible unblocked child** of *this* epic — not the globally lowest issue outside the epic — once the epic is the next global candidate.
- **Nested parents**: if that child also has children, expand again until a leaf.
- While a child is still open/eligible: **do not** claim or mark Done on the epic/parent; work only the leaf child.
- **Preferred id is an epic** (`/solve TEAM-100` where TEAM-100 is parent): expand into its lowest eligible child (do not implement TEAM-100). If all children already terminal → mark epic Done (rollup) and pick the next global candidate.
- **Preferred id is a child**: use that child if eligible/unblocked (no need to expand upward).
- When starting a child from an epic: note in the **leaf** start comment `via epic TEAM-100`.

#### Epic rollup (all children complete)

When **every** child of an epic is in a terminal state (Done / Canceled / Cancelled / Duplicate / Completed). **In Review is not terminal.**

1. Comment on the epic: all children complete; closing epic as rollup (list child ids if short).
2. Set epic state **Done**.
3. Do **not** run implement/git for the epic shell.
4. Do **not** count the epic toward `SOLVE_COUNT_MODE` (it is free hygiene during selection or after the last child closeout).
5. Continue selecting the next global candidate.

**In Review is not terminal.** After a leaf closeout the leaf is In Review, so **do not** rollup the epic in Phase 8. `/prb` / `/tidy` roll up after `main`.

### 2F. Inspect before claiming

For the **resolved leaf** issue (after epic expansion), load full issue: description, acceptance criteria, code map, drift check, comments, relations, labels, and **parent epic id/title** when expanded.

If the ticket is too thin to implement safely, either:

- Do minimal research and implement if scope is still clear, or
- Skip with reason and take the next number (do not stall the whole run on a placeholder ticket unless nothing else is eligible)

### 2G. None eligible

End the batch loop (do not invent work). Report in Phase 9:

- Team/project used
- How many already solved this run (if any)
- Guidance path (if S0 ran), execution order, and supersession skips
- Candidates considered and why skipped (blocked / in progress other / wrong state / epic with no eligible children / abandoned platform)

If this is the first selection and nothing is eligible → stop the whole `/solve` with that report. Note when the board is “drained of implementable work” but still has open tickets only because they were **guidance-skipped** as obsolete.

---

## Phase 3 — Linear start

Only for the **leaf** issue we are about to implement (not for skipped candidates):

1. Confirm the leaf is **unclaimed** (see [`references/multiplayer-linear.md`](references/multiplayer-linear.md)). If `/solve all` / `fast` and foreign live claims exist on this project, **do not drain** — skip to unclaimed leaves or stop and report.
2. Assign to **me** if unassigned (when API allows).
3. Set state **In Progress** on the leaf (not the parent epic, unless the epic is being rollup-closed per 2E).
4. **Required claim + start comment** on the leaf. First line MUST be:
   `claimed-by: <bot-or-cli> · session <id> · worktree <cwd> · run <RUN_ID|sequential>`
   Then: why this order, epic expansion path if any, plan in 3–6 bullets, implement→review + verify, delivery = **`origin/dev` then In Review (not Done)**, batch guidance path when S0 ran.
5. **Re-read the issue immediately.** If a different run's `claimed-by:` is newer, abort this leaf (do not code) and pick another.
4. **Required** brief comment on the parent epic when expanded: `Starting child TEAM-205 (lowest eligible) under this epic.` (skip if no parent).
5. When S0 **skips/cancels** an obsolete open ticket as full supersede: **one** comment on that ticket (superseded by …; not implemented). Do not spam other skips.

Do **not** comment on issues we only evaluated and skipped for ordinary ordering reasons.

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

---

## Phase 5 — Implement via /implement

**Do not implement the issue yourself.** Run the full implement skill as a nested procedure.

### 5A. Load implement procedure

1. `read_file` on `IMPLEMENT_SKILL_MD` (full skill body).
2. `read_file` on `CUSTOM_IMPL_INSTRUCTIONS`.
3. Resolve persona paths as the implement skill documents (under implement’s shared personas / skill dir).
4. Resolve `memory.py` from **`IMPLEMENT_SKILL_DIR/scripts/memory.py`**, not from the solve skill dir.

### 5B. Build the implement task description

Construct a single description string for the implement loop:

```markdown
## Linear issue
- Id: <TEAM-123>
- Title: <title>
- URL: <url>
- Parent epic (if expanded): <TEAM-100 — title> or none
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

## Solve delivery constraints (hard)
- Work only on the current git branch: <issue-branch>
- Scope is this leaf issue only — do not implement the full parent epic
- Do not push, open PRs, merge to dev/main, or update Linear state
- Do not discard unrelated dirty files
- Prefer not to commit; leave the tree ready for the solve orchestrator to commit after verify
- Smallest complete change meeting **current** (possibly re-scoped) acceptance criteria
- Runtime proof: follow `$SOLVE_SKILL_DIR/../docs/prove-it-works.md` (in-scope classes). Matrix green is not enough.

## Custom solve implement instructions
<verbatim contents of custom-implement-instructions.md>
```

### 5C. Effort

Use CLI `--effort N` if valid (1–5). Else default from custom instructions (currently **5** — maximum rigor).

### 5D. Run the implement skill end-to-end

Follow **every** required step of the implement skill for this description and effort, including:

- Tool-call discipline and todo scaffold (implement phase ids)
- Persona injection and bracketed `description` tags
- Memory retrieval / flush via implement’s `memory.py`
- Implementer spawn → review → fix → re-review until **0 open issues**
- Stalemate escalation to the user when needed
- Scratch summary/review files under the implement skill’s scratch_dir rules

**Orchestrator rule (solve):** while the implement loop is active, you **must not** use `write` / `search_replace` / shell to modify application source for the issue. Only implementer/reviewer subagents change product code. You may still run read-only tools, Linear updates that don’t close the issue, and git status/diff inspection.

If implement fails (subagent hard-fail, unrecoverable):

- **Integer `N` mode:** **halt the batch** — leave Linear **In Progress**, do not merge to `dev`, do not start the next issue, report in Phase 9.
- **`all` mode:** leave Linear **In Progress** (or **Blocked** if human/external), do not merge to `dev`, append to `FAILED`, cascade-skip dependents, **continue** selecting the next independent eligible leaf. Only escalate/stop the whole run if the user aborts or a stalemate needs human input on the only remaining work.

### 5E. After implement reports clean

1. Read the implement summary file (before implement cleanup if still present; otherwise reconstruct from `git diff` / status).
2. Confirm working tree changes match the issue scope.
3. Proceed to Phase 6 (verify). Do **not** merge to `dev` yet.

---

## Phase 6 — Verify

**Authority:** [`../docs/prove-it-works.md`](../docs/prove-it-works.md). **Read that file.**

Order:

1. Issue **Verification** / acceptance criteria / **Runtime proof** section
2. Repo `AGENTS.md` / README matrix for the touched package(s)
3. **Runtime proof** when in-scope (user-visible, auth, billing, public API, schema, shared helper): drive the real path; visual parity if the ticket names a reference; blast-radius run if shared/schema/auth
4. Typical matrix (adjust per repo): `typecheck`, relevant tests, `build` when AGENTS requires it

Matrix green is **necessary, not sufficient** for in-scope work. “Not performed” is a **fail**.

Fix failures **by resuming the implementer** (same implement loop rules: spawn/resume implementer, not by editing yourself), then re-verify. Do not merge to `dev` or In Review if required checks **or** in-scope runtime proof fail.

### Commit (after verify passes)

On the **issue branch**, stage **only** relevant paths and commit:

```text
TEAM-123: short imperative summary
```

Use HEREDOC for the commit message. If the implementer already committed clean issue-only commits, add a follow-up commit only if needed; do not amend published commits (none should be pushed yet).

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
- **Then Phase 7B — verify/build/security on `dev`, then push `origin/dev`.**

### Phase 7B — Build, security checks, push `origin/dev`

After the issue branch is merged into local `dev`:

1. Ensure `origin/main` is still merged into local `dev` (re-fetch if trunk moved).
2. Run repo-required **build + verify + security** checks (from `AGENTS.md` / package scripts / documented scanners). Fail → fix via implement loop; **do not push**.
3. `git push -u origin dev` (create `origin/dev` if missing). No user approval required.
4. Record the `origin/dev` SHA in the Linear closeout comment.

If merge to local `dev` fails, do not push and do not mark Linear Done; fix or report.
If checks fail, do not push; leave leaf In Progress with a failure comment.

---

## Phase 8 — Linear closeout

After verified merge to local `dev` for the **leaf** work issue:

1. **Required** completion comment on the leaf:
   - Summary of code changes (paths)
   - Implement effort + review rounds (if known)
   - Verification commands + results **and** runtime-proof evidence from [`../docs/prove-it-works.md`](../docs/prove-it-works.md) (driven path + observed state; visual reference or `n/a`; blast-radius fact or `n/a`)
   - Local branches: issue branch name + **merged into local `dev`**
   - Explicit: **pushed to `origin/dev`** after checks; **no** merge to `main`/prod (unless user explicitly approved a separate `/prb`)
   - Parent epic reference if this was an expanded child
   - Follow-ups filed or recommended
2. Set state **In Review** on the **leaf** if the team has that status; otherwise leave **In Progress**. Never **Done** here (`/prb` owns Done after `main`).
3. If this leaf had a parent epic:
   - Re-list children of the parent.
   - **In Review is not terminal.** **Required** brief progress comment on the epic (`Child TEAM-205 In Review (origin/dev); remaining: …`). Do **not** mark the epic Done.
   - Epic rollup (comment + Done) only if **every** child is already terminal (Done/Canceled/Duplicate). A leaf just set to In Review keeps the epic open. `/prb` / `/tidy` roll up after `main`.
4. Brief comments on **related** issues only if this clearly unblocks them (optional but preferred when obvious).

If verification failed or dev merge failed:

- Leave leaf **In Progress**, or set **Blocked** with reason when blocked on human/external input.
- **Required** failure comment on the leaf explaining what failed. In **integer `N`** multi-issue mode, note that the batch halted. In **`all`** mode, note that this leaf failed and the drain **continues** with other independent eligible issues.
- Do **not** mark Done; do **not** close the parent epic.

---

## Phase 9 — User reply (concise)

### Single issue (`SOLVE_COUNT_MODE = 1` and one attempt)

```markdown
**Solved:** [TEAM-123](url) — <title>
**Via epic:** [TEAM-100](url) — <epic title>   <!-- omit if not expanded -->
**Linear:** leaf In Review (`origin/dev`); epic <left open | rollup Done only if all children terminal>
**Branch:** `<issue-branch>` → local `dev` → **`origin/dev` @ sha** (after checks)
**Main/prod:** not shipped (needs explicit approval)
**Implement:** effort N · review rounds R · 0 open review issues
**Verify:** <matrix commands + pass/fail> · **Runtime:** <driven path + observed state | n/a out-of-scope>
**Push:** `origin/dev` @ sha (after checks) · **main/prod:** not shipped (needs explicit approval via `/prb`)
**Notes:** <one line if needed>
```

### Multi-issue batch (`N > 1` or `all`, sequential)

**Only emit this after the [drain gate](#drain-gate-before-phase-9) when mode is `all`.** Never use Remaining to excuse an incomplete `all` drain.

```markdown
**Batch:** solved K of target <N|all> · attempted A · failed F · skipped S
**Mode:** /solve <N|all> · effort E
**Team/project:** <resolved>
**Guidance:** `<abs path to guidance.md>` · canonical: … · abandoned: …
**Order:** TEAM-… → TEAM-… (migrations/foundations first when relevant)
**Drain (all only):** verified — no eligible unblocked implementable leaves remain

### Solved
1. [TEAM-123](url) — <title> · branch `…` → local `dev` · verify pass
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
**Push:** `origin/dev` @ sha (after checks) · **main/prod:** not shipped (needs explicit approval via `/prb`)
```

### Fast batch (`FAST_MODE`)

**Only emit after F6 + drain gate when mode is `all`.**

```markdown
**Batch (fast):** solved K of target <N|all> · attempted A · failed F · skipped S
**Mode:** /solve <N|all> fast · concurrency C · effort E · run <RUN_ID>
**Team/project:** <resolved>
**Guidance:** `<abs path to guidance.md>` · canonical: … · abandoned: …
**Waves:** W · max parallelism used: P
**Cleanup:** worktrees removed K · branches deleted K · remaining solve worktrees: 0
**Drain (all only):** verified — no eligible unblocked implementable leaves remain

### Solved
1. [TEAM-123](url) — <title> · merged → local `dev` · verify pass

### Skipped (superseded / abandoned platform)
- [TEAM-67](url) — reason

### Failed
- [TEAM-125](url) — reason (left In Progress; dependents skipped: …)

### Remaining
- Implementable eligible still open: **none** (required for all after drain gate)
  <!-- integer N may list leftovers and suggest re-run -->
**Push:** `origin/dev` @ sha (after checks) · **main/prod:** not shipped (needs explicit approval via `/prb`)
```

If `all` drained implementable work: state clearly that **Drain verified: no eligible unblocked implementable issues remain** (mention guidance-skipped obsolete, blocked, or other-assignee tickets separately if still open). **Do not** tell the user to re-run `/solve all` to finish work that this run should have continued.

---

## Blocked definition (quick reference)

| Signal | Action |
|--------|--------|
| `blockedBy` open issue | Skip (no comment) |
| State/label Blocked | Skip (no comment) |
| Needs missing secrets / human decision | Skip (no comment) unless already claimed → then Blocked + comment |
| In Progress + other assignee | Skip (no comment) |
| Done/Canceled/Duplicate | Skip (no comment) |
| In Review | Skip (no comment) — waiting on `/prb` / `main` |
| Guidance full-obsolete / abandoned platform | **Skip implement**; one Linear comment; prefer Cancel/Duplicate when high confidence |
| Parent/epic with eligible children | **Expand** → lowest eligible child (do not implement parent) |
| Epic, all children terminal | Comment + mark epic **Done** (rollup); continue; no solve-count |
| Epic, no eligible children but some non-terminal | Skip epic (no comment); continue |

---

## Linear issue management

Canonical policy for status, assignee, and comments under `/solve`.

### Status & assignee (when we touch Linear)

| Moment | Issue | Status | Assignee | Comment? |
|--------|-------|--------|----------|----------|
| Skipped during selection (blocked, wrong state, other assignee, thin ticket, epic not rollup-ready) | candidate | unchanged | unchanged | **No** |
| Epic rollup — all children terminal | epic | → **Done** | unchanged | **Yes** (rollup note) |
| Phase 3 start work | **leaf** | → **In Progress** | → **me** if unassigned | **Yes** (start plan) |
| Phase 3 (expanded child) | parent epic | unchanged | unchanged | **Yes** (starting child …) |
| Phase 5–7 (implement/verify/merge) | leaf | stays In Progress | stays | no routine mid-flight comments |
| Phase 8 success | **leaf** | → **In Review** (never Done) | unchanged | **Yes** (completion evidence; `origin/dev`) |
| Phase 8 success, parent epic | parent epic | unchanged | unchanged | **Yes** (child In Review; remaining …) |
| Phase 8 / implement failure | **leaf** | stays **In Progress**, or → **Blocked** if human/external | unchanged | **Yes** (failure reason) |
| Related issues unblocked | related | unchanged unless policy says otherwise | unchanged | optional short note |

### Comment rules (summary)

- **Yes, comment** on every issue we **claim** (start) or **complete** (In Review) or **fail** after claiming, plus epic start/progress/rollup notes tied to that work.
- **No, do not comment** on every issue we merely scan and skip while hunting for the next eligible leaf.
- **No spam**: one start comment and one completion (or failure) comment on the leaf is enough; avoid play-by-play during implement/review.
- Implementer/reviewer subagents **must not** update Linear state or close issues (solve orchestrator owns Linear).
- Epic Done from rollup is **not** a substitute for implementing a leaf, and **does not** count toward `/solve N`.

### What we are *not* doing today

- No comments on the long tail of ordinary “not next” skips during selection.
- No mass status changes on low-confidence platform inference (escalate instead).
- High-confidence full-obsolete supersedes may get **one** comment and Cancel/Duplicate.
- Closeout should include `origin/dev` SHA after push; still no `main`/prod ship by default.
- No marking parent Done when only *some* children finished.

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
- Treating `/solve` as unlimited when the user did not pass `all` (default is **1**)
- Treating `/solve all` as a soft batch of ~5 (or any fixed K) instead of a full drain
- Confusing `--effort N` or `--concurrency N` (or default effort **5** / concurrency **4**) with solve count `N`
- Emitting Phase 9 for `all` while eligible unblocked leaves still exist (“re-run to continue”)
- Skipping the **drain gate** Linear re-query in `all` mode
- Freezing on the initial S0 inventory in `all` mode without refill after merges/closeouts
- Pre-queuing without S0 analysis when multi-issue guidance is required
- Starting the next issue before the current one is merged to local `dev` and closed out (**sequential**; fast: do not start dependents before hard deps are merged to `dev`)
- Continuing a sequential **integer-`N`** batch after an implement/merge failure without user direction
- Hard-stopping sequential **`all`** after one independent failure while other eligible leaves remain
- Implementing the issue **yourself** instead of the implement loop
- Ignoring `custom-implement-instructions.md` or `guidance.md`
- Using capital-`D` **`Dev`** (or any other casing) instead of lowercase **`dev`** for the integration branch
- Creating a new branch named `Dev`
- Skipping build/security checks before `git push origin dev`
- Merging to `main` or deploying/prod from `/solve` (that is `/prb` + explicit approval)
- Leaving a repo without long-lived `dev` on local + `origin`
- Implementing on `main` directly
- Letting `dev` fall behind `main`
- Marking **Done** from `/solve` (Done is `/prb` after `main`; exception: epic rollup when all children terminal)
- Marking In Review without verification or without push to `origin/dev`
- Treating typecheck/tests/build as proof for user-visible, auth, billing, API, schema, or shared-helper changes ([`../docs/prove-it-works.md`](../docs/prove-it-works.md))
- Writing “browser smoke not performed” and still In Review
- Starting a second `/solve all` / `fast` drain on a project that already has foreign live `claimed-by:` comments
- Coding a leaf after a failed claim re-read (CAS miss)
- Committing unrelated dirty files or secrets
- Expanding into a rewrite when the ticket asked for a narrow fix (unless guidance re-scope requires platform rewrite)
- Silently skipping Linear updates on issues we claimed or completed
- **Fast:** starting workers before guidance.md + graph.json exist
- **Fast:** worker merges to `dev`, pushes, or sets Linear Done
- **Fast:** leaving worktrees or issue branches after merge/fail (cleanup is mandatory)
- **Fast:** using only `git worktree remove` for tool-created worktrees — use `grok worktree rm --force`
- **Fast:** exceeding concurrency 8 or spawning separate `grok` CLI processes (v1)
- **Fast:** starting a dependent while its hard dep is only done in a worktree, not yet on local `dev`

---

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/issue` | Files thorough tickets only |
| `/implement` | Implement→review loop only; no Linear pick / no git delivery |
| `/solve` | Selects Linear issue(s) + implement loop + local `dev` + **`origin/dev` push after checks**; count via `/solve [N\|all]` (default 1); optional **`fast`** parallel orchestrator |
| `/execute-plan` | Parallel worktree implement from a design-doc PR DAG (not Linear pick/closeout) |
| `/prb` | Promotes already-pushed `origin/dev` → PR → main/prod; **`origin/dev` is automatic**, **main/prod needs explicit user approval** |
| GSD | Broader delivery (push/PR/deploy); do **not** assume those defaults here |

---

## Customizing implement behavior

Paths below are **skill-relative** (resolved from `SOLVE_SKILL_DIR` = directory containing this skill’s `SKILL.md`, i.e. the skill install location — not a hard-coded home path).

Edit:

```text
references/custom-implement-instructions.md
# absolute: $SOLVE_SKILL_DIR/references/custom-implement-instructions.md
```

That file is the supported extension point for coding policy under `/solve`. Prefer editing it over forking the entire implement skill. Standalone `/implement` is unchanged.

Batch guidance (multi-issue order, supersession, platform direction) lives in:

```text
references/batch-guidance.md
references/batch-guidance-template.md
references/graph.schema.json
# absolute: $SOLVE_SKILL_DIR/references/<file>
```

Fast-mode orchestration (waves, worktrees, cleanup) lives in:

```text
references/fast-mode.md
references/architecture-guidance-template.md
# absolute: $SOLVE_SKILL_DIR/references/<file>
```

`architecture-guidance-template.md` is a **fast supplement**; `guidance.md` from batch guidance is the **authority** for platform direction and supersession.

---

## Linear or git failure

- **Linear auth fails**: report re-auth needed; do not pretend coordination. Optionally still implement a user-named local issue only if they insist.
- **Cannot ff main / dev conflicts**: stop with exact commands and conflict list; do not force.
- **Implement skill missing**: report path resolution failure; stop.
- **Batch guidance / fast package fails**: stop before claim/workers; report inventory/graph/direction problems; do not spawn parallel implementers or implement obsolete stack work.
- **Fast worktree cleanup fails**: retry `grok worktree rm --force` once; include leftover paths in Phase 9; do not pretend remaining worktrees is 0.
- **Push requested later**: user may run a separate ship step; this skill’s default remains local-only.
