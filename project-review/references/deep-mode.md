# /project-review deep — Orchestrator protocol

Canonical procedure when `MODE=deep`. Sequential single-agent **fast** mode ignores this file except for the shared local-candidate preference.

Parent skill: [`../SKILL.md`](../SKILL.md)  
Guidance procedure: [`review-guidance.md`](review-guidance.md)  
Guidance template: [`review-guidance-template.md`](review-guidance-template.md)  
Candidate package: [`issue-candidates.md`](issue-candidates.md)  
Inventory schema: [`inventory.schema.json`](inventory.schema.json)  
Worker prompt: [`worker-prompt.md`](worker-prompt.md)

---

## Purpose

Run an **exhaustive** project review using:

1. A full **inventory** of review units (routes, UI views, APIs, components, auth, jobs, …)
2. A **review guidance package** workers must obey
3. Parallel **read-only review workers** until coverage is complete
4. **Local file-first** issue candidates (`issue-candidates/`) cleaned offline
5. **One** Linear board snapshot early + **one** Linear publish pass at the end from `final/` only

Deep does **not** stop when “enough high-signal findings exist.” It stops when every inventory unit is terminal **and** the local candidate package is cleaned (then optionally filed).

---

## Constants

| Name | Value |
|------|--------|
| `DEFAULT_CONCURRENCY` | `4` |
| `MAX_CONCURRENCY` | `8` |
| `--concurrency M` | Clamp to `1..MAX_CONCURRENCY` |
| Soft worker timeout | ~45–60 minutes; then kill, mark slice failed, re-queue incomplete units |
| Worktrees | **Not used** (discovery only; shared workspace) |
| Linear during discovery | **Forbidden** except Phase D1b snapshot |
| Linear publish source | `issue-candidates/final/*.md` only |

---

## Phase map

```text
Phase 0–1   Bootstrap + intended state (SKILL.md)
Phase D0    Full project inventory
Phase D1    Write review guidance package (scratch dir)
Phase D1b   ONE Linear board snapshot → board-snapshot.json
Phase D2    User-visible coverage plan (non-blocking)
Phase D3    Worker loop → local candidate files
Phase D4    Coverage gate
Phase D5    Local cleanup (dedupe, board_match, pin, final/)
Phase D6    Dependency graph on final/ only
Phase D7    Linear file pass OR --draft stop
Phase D8    Handoff
```

After D5–D6, Phase D7 replaces the old “Phase 7 file while inventing” mental model. Board matching is offline against the snapshot (see [`board-sync.md`](board-sync.md)).

---

## Scratch layout

```bash
RUN_ID=$(python3 -c "import uuid; print(uuid.uuid4().hex[:8])")
scratch_dir="${TMPDIR:-/tmp}/grok-$(id -u)/project-review-${RUN_ID}"
mkdir -p \
  "$scratch_dir/issue-candidates/by-route" \
  "$scratch_dir/issue-candidates/by-api" \
  "$scratch_dir/issue-candidates/by-component" \
  "$scratch_dir/issue-candidates/cross-cutting" \
  "$scratch_dir/issue-candidates/_inbox" \
  "$scratch_dir/issue-candidates/_merged" \
  "$scratch_dir/issue-candidates/final" \
  "$scratch_dir/workers" \
  && chmod 700 "${TMPDIR:-/tmp}/grok-$(id -u)" "$scratch_dir"
echo "$scratch_dir"
```

| Path | Role |
|------|------|
| `guidance.md` | Authority for all workers |
| `inventory.json` | Full review unit list |
| `coverage.json` | Per-unit status, slice assignment |
| `board-snapshot.json` | One-shot open Linear issues for offline dedupe |
| `state.json` | Run metadata, concurrency, phase, worker task ids |
| `workers/<slice_id>.md` | Per-worker coverage report |
| `issue-candidates/**` | Working + final ticket package (see issue-candidates.md) |

Record absolute `SCRATCH_DIR` in `state.json` and every worker prompt. Do not rely on env vars surviving across shells.

---

## Phase D0 — Full project inventory

Build an exhaustive list of **review units**. Prefer mechanical discovery (glob/grep) over sampling.

### Kinds

| `kind` | Discovery sources |
|--------|-------------------|
| `route` | `app/**/page.tsx`, `pages/**`, route configs, native navigators |
| `ui_view` | Major modals/drawers/tabs owned by a route (when clearly separate) |
| `component` | Shared UI / design-system package exports; high-churn app shell components |
| `api` | `app/api/**`, `route.ts`, `"use server"` actions, tRPC/REST routers |
| `data` | Schema/migrations/loaders tightly coupled to product surfaces |
| `auth` | Middleware, session helpers, role gates |
| `job` | Crons, queues, webhooks |
| `nav` | Sidebar/tab/config entries claiming surfaces exist |
| `docs_intent` | PRD/nav claims not yet represented as routes (completeness gaps) |

### Completeness rule

Every product-facing `route`, every registered `api`/server action used by the app, every shared component package module imported from product UI, and every nav-declared surface **must** appear. Group tiny pure helpers under a parent unit rather than inventing hundreds of micro-units.

### Scope

| Flag | Behavior |
|------|----------|
| Default | Whole project inventory |
| `--scope-only` | Inventory limited to SURFACE bias (paths/features user named) |
| Surface bias without `--scope-only` | Still inventory whole project; mark bias area `priority: primary`, rest `secondary` — **both reviewed** |

### Procedure

1. Package/app map (`apps/*`, `packages/*`, monorepo roots).
2. Route map per app.
3. API / server-action map.
4. Shared UI inventory.
5. Auth/middleware boundaries.
6. Jobs/webhooks if present.
7. Nav config vs routes (dead-link seeds).
8. Merge Phase 1 intended-state journeys → each unit `must_review: true` in deep.
9. Assign stable `unit_id`s (e.g. `apps-web-route-dashboard`, `apps-web-api-projects-create`).
10. Write `inventory.json` conforming to [`inventory.schema.json`](inventory.schema.json).

**Gate:** refuse workers if zero product units (report greenfield/empty honestly).

Optional: spawn 1–2 read-only explore subagents to accelerate route/API enumeration; orchestrator owns the merged inventory.

---

## Phase D1 — Review guidance package

Full procedure: [`review-guidance.md`](review-guidance.md).

1. Fill [`review-guidance-template.md`](review-guidance-template.md) → `guidance.md`.
2. Initialize `coverage.json` from inventory (all `pending`).
3. Plan **slices** (package + route prefix / API domain / shared UI). Target ~5–25 units per slice.
4. Write initial `state.json` (`status: planning`, concurrency, paths).
5. Create empty `_merged/index.json` structure.

**Gate:** do not start workers until `guidance.md`, `inventory.json`, and `coverage.json` exist.

---

## Phase D1b — Board snapshot (one Linear fetch)

1. Page open/actionable Linear issues for team + project **once**.
2. Capture: id, identifier, title, state, priority, labels, parent, url, short body excerpt, code-map paths if present.
3. Write `board-snapshot.json` (+ optional `board-snapshot.md`).
4. **Stop using Linear** until Phase D7.

If Linear fails: set `board_snapshot: null` in state; continue offline; publish may be draft-only.

Workers never call Linear. Orchestrator never re-lists the board per candidate during D3–D5.

---

## Phase D2 — Coverage plan (user report, non-blocking)

```text
Deep plan: U units · S slices · concurrency C · run <RUN_ID>
Inventory: N routes · M APIs · K components · …
Scratch: <SCRATCH_DIR>
Board snapshot: yes|no
Proceeding with workers (no confirmation needed).
```

Proceed unless inventory empty or user asked for dry-run only.

---

## Phase D3 — Worker loop

### Launch

`spawn_subagent`:

| Param | Value |
|-------|--------|
| `subagent_type` | `explore` preferred for pure code walk; `general-purpose` if browser/live tools needed |
| `capability_mode` | `read-only` when available (explore default) |
| `isolation` | `none` (shared workspace — workers write under SCRATCH_DIR only) |
| `background` | `true` |
| `description` | `[project-review-deep] <slice_id>` |

**Do not** use worktree isolation for review workers (no app code edits; worktrees add cost without benefit).

Prompt: fill [`worker-prompt.md`](worker-prompt.md) with absolute paths and unit ids.

### Ready-queue

```text
while any unit pending|failed (retry budget left):
  ready = slices with pending units, not in_progress, under concurrency
  spawn up to CONCURRENCY workers
  wait_any(worker task ids)
  for each completed:
    validate workers/<slice_id>.md exists
    ingest issue-candidates/_inbox/<slice_id>/ into index
    mark units reviewed | blocked_auth | failed
    re-queue incomplete units (split slice if worker under-reviewed)
Coverage gate before D5
```

### Worker rules (hard)

1. Read `guidance.md` fully.
2. Review **every** assigned unit (empty findings OK; must mark unit reviewed with notes).
3. Write candidates **only** under `issue-candidates/` (see issue-candidates.md).
4. Write coverage report to `workers/<slice_id>.md`.
5. **Never** Linear MCP, never create tickets, never edit application source.
6. Run full deep lenses (`[F]` + `[D]`) on assigned units.

### Progress

Keep a visible tally: `Coverage 42/180 · 3 workers live · candidates 27 · scratch …`.

### Live walk

If `LIVE_URL` or local app is available:

- Schedule live-walk slices for routes (deep = all reachable routes; auth-gated as far as tools allow).
- Live evidence writes candidates like code findings; units still need code review (live does not replace inventory units).

---

## Phase D4 — Coverage gate

Deep discovery is incomplete until every inventory unit has a **terminal** status:

| Status | Meaning |
|--------|---------|
| `reviewed` | Worker completed lenses; zero or more candidates |
| `blocked_auth` | Could not access live path; **code-only** review done |
| `out_of_scope` | Explicitly excluded (`--scope-only` residual, non-goal, test fixture) with reason |
| `failed` | Must retry or split until resolved; not terminal |

**Forbidden:** ending deep because “we have enough tickets” or context is long while units remain `pending`.

After gate: Phase D5 (still no Linear create).

---

## Phase D5 — Local cleanup

Full rules: [`issue-candidates.md`](issue-candidates.md).

1. Ingest all `_inbox` + `by-*` candidates into `_merged/index.json`.
2. Near-dupe merge (same surface + root cause / same primary path).
3. Offline `board_match` against `board-snapshot.json` only.
4. Signal filter (drop pure nits; deep keeps well-formed P2).
5. Code pin + taste conversion; expand to full issue-template.
6. Write publish set to `issue-candidates/final/*.md` (`status: ready_to_file`).
7. Quality gate every final body (same bar as issue-template.md).

Optional: spawn pin workers over batches of candidate files (read-only + write only under scratch).

`--draft` / Linear unavailable: stop after D5–D6; handoff package path; **not filed**.

---

## Phase D6 — Dependency graph

Apply [`dependency-ordering.md`](dependency-ordering.md) to **final/** only.

Record planned `blocked_by` as **candidate ids** in `_merged/index.json`. Map to Linear ids after create.

Filing order: foundations → features → polish/content/a11y.

---

## Phase D7 — Linear publish pass

Full policy: [`linear-filing.md`](linear-filing.md).

1. Create epic (unless `--no-epic`).
2. Create leaves **only** from `issue-candidates/final/*.md` in filing order.
3. Set relations from index (`relatedTo` board matches, `blockedBy` after ids exist).
4. Write `linear_id` / url back into `_merged/index.json`.
5. On partial failure: continue remaining finals; handoff lists disk recovery path; **do not delete scratch**.
6. On **full success** (verified): apply [Scratch directory lifecycle](#scratch-directory-lifecycle-mandatory).

**Never** create from `_inbox` or uncleaned `by-*` dumps.

Do not re-crawl Linear mid-publish. On unexpected duplicate API error: mark candidate, continue.

### Scratch directory lifecycle (mandatory)

| Outcome | Scratch dir (`project-review-<RUN_ID>`) |
|---------|----------------------------------------|
| **All** intended `final/` leaves filed to Linear (create succeeded; ids recorded) | **Delete** entire `SCRATCH_DIR` after verify |
| `--draft` / not filed | **Keep** — path required for later publish |
| Linear unavailable / auth fail | **Keep** |
| Partial file (some finals failed) | **Keep** — remaining `final/` is the re-file source |
| Discovery only, never reached D7 | **Keep** until filed or user discards |

**Simple rule: filed in Linear → delete temp. Not in Linear → keep temp.**

**Verify before delete (all required):**

1. Every intended publish final has `status: filed` and non-null `linear_id` in the index (or was never in the publish set by design — e.g. filtered before `final/`).
2. Unfiled bodies remaining in `final/` → **do not delete**.
3. Epic created when required (unless `--no-epic`).
4. Handoff already has epic + leaf identifiers/URLs (capture **before** deleting).

```bash
# Only after verify — entire run package, not just final/
rm -rf "$SCRATCH_DIR"
```

Do **not** delete parent `grok-$(id -u)/` (other runs may exist).  
Do **not** delete on draft, partial, or unverified publish.

---

## Phase D8 — Handoff

Use [`handoff-template.md`](handoff-template.md) deep sections:

- Coverage counts (units terminal breakdown)
- Filed epic + leaves (ids/URLs)
- Duplicates skipped (from offline board_match)
- Suggested `/solve` commands
- **Scratch:** absolute path if **kept**; or note `deleted after successful Linear file` if removed

**Stop.** Do not implement or auto-run `/solve` unless user asks.

---

## Anti-patterns (deep-specific)

- Stopping deep early with pending inventory units
- Single-agent “skim” claiming exhaustive deep coverage
- Workers calling Linear or filing tickets
- Re-listing Linear for every candidate during cleanup
- Creating Linear issues from raw `_inbox` files
- One mega “fix everything” ticket
- Using worktrees for review workers
- Treating coverage as “one ticket per unit” (reviewed with zero findings is success)
- Flooding final/ with unpinned nits
- Asking the user to list bugs before inventory/workers
- **Deleting scratch while any intended final is unfiled** (draft, partial, or failed publish)
- **Leaving scratch forever after a fully verified Linear publish** (should delete)

---

## Relation to fast mode

| | Fast | Deep |
|--|------|------|
| Inventory | Primary journeys only | Exhaustive units |
| Workers | Optional / none | Required orchestrator loop |
| Guidance package | Optional light scratch | Required |
| Coverage gate | Soft (high-signal done) | Hard 100% terminal |
| Candidates on disk | Preferred small folder | Required `issue-candidates/` tree |
| Linear | Snapshot + file from finals preferred; may be lighter | Snapshot once + publish from final/ only |
