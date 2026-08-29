# Batch guidance — supersession, platform direction, and order

Canonical procedure for building the **batch guidance package** used by:

- Sequential `/solve N` when `N ≥ 2`
- Sequential `/solve all`
- Any `/solve … fast` run (replaces/extends Phase F2 “architecture only” thinking)

Parent skill: [`../SKILL.md`](../SKILL.md)  
Template: [`batch-guidance-template.md`](batch-guidance-template.md)  
Graph schema: [`graph.schema.json`](graph.schema.json)

---

## Why this exists

Lowest-issue-number order is wrong when:

1. An **older open** ticket assumes stack A (e.g. ClickHouse, Convex).
2. A **newer open** ticket (or repo docs) declare stack B (e.g. Neon Postgres) as canonical.
3. Feature tickets would be **rewritten** if implemented on A before the migration to B.

**Done** tickets are closed history: do not re-implement them. A newer ticket may **selectively override** their code. The dangerous case is **open** tickets that still target an abandoned direction.

---

## When to run (Phase S0)

| Invocation | Run full S0? |
|------------|----------------|
| `/solve` / `/solve 1` | Only if preferred issue has explicit `## Supersedes` / stack conflict, or user constraints demand it |
| `/solve N` (`N ≥ 2`) | **Yes**, before first claim |
| `/solve all` | **Yes**, before first claim |
| `/solve` with `SCOPE` (implicit `all`) | **Yes**, on the in-scope leaf set only |
| Any `fast` | **Yes** (F1 inventory + this analysis before workers) |
| `/identify` | **Thin** subset only — tag, skip full-obsolete, promote migrations when a conflict exists. No scratch files. No Linear writes. See identify `references/guidance.md`. |

If S0 is not required, sequential mode may keep pure lowest-number selection.

---

## Scratch layout

```bash
RUN_ID=$(python3 -c "import uuid; print(uuid.uuid4().hex[:8])")
scratch_dir="${TMPDIR:-/tmp}/grok-$(id -u)/solve-batch-${RUN_ID}"
mkdir -p "$scratch_dir" && chmod 700 "${TMPDIR:-/tmp}/grok-$(id -u)" "$scratch_dir"
```

| File | Role |
|------|------|
| `guidance.md` | Human authority for the run (implementers **must** read) |
| `inventory.json` | Tagged candidates |
| `graph.json` | Order, deps, supersedes, skips (schema-conformant) |
| `state.json` | solved / skipped / failed / paths |

Fast mode may use the same dir naming (`solve-batch-*` or `solve-fast-*`) but **must** produce equivalent guidance fields.

---

## Design principles

1. **Newest explicit direction + repo truth** beat older open tickets on platform/stack.
2. **Migration / platform foundation before** features that depend on the abandoned stack.
3. **Selective override** only inside declared scope.
4. **Done is closed** — override via newer tickets, do not reopen Done issues to “finish” them.
5. **Lowest number is the default tie-breaker** among independent, non-conflicting leaves.
6. **Escalate** when two open tickets claim conflicting “new” architectures and repo truth is weak.

---

## Step A — Inventory

1. Linear inventory fetch in [`eligibility.md`](eligibility.md) (team + project + state per actionable status; slim fields; page each state).
2. If `SCOPE` is set, keep `issue_in_scope` only ([`eligibility.md`](eligibility.md) Scope filter).
3. Apply eligibility (Phase 2B), blocked (2D), epic expansion (2E) → **leaf** set.
4. Optionally sample **recent Done** issues that share keywords/paths with open leaves (override context only).
5. Read repo architecture anchors when present:
   - `docs/architecture.md`, `docs/prd.md`, root `README.md`
   - Deferred/superseded plan docs under `docs/plans/`
   - Package layout (`apps/`, `packages/`) for actual runtime

Record for each leaf: id, title, url, state, assignee, parent epic, blockedBy, relatedTo, body excerpt, primary_paths (from code map / inference).

---

## Step B — Tag each open leaf

### Platform tags (non-exhaustive)

`neon`, `postgres`, `clickhouse`, `convex`, `railway`, `vercel_blob`, `redis`, `unknown`, plus product-specific names found in repo docs.

### Issue class

| Class | Signals |
|-------|---------|
| `migration` | migrate, cut over, replace stack, remove X, move to Y |
| `foundation` | ingest pipeline, schema foundation, auth core, shared SDK contract |
| `feature` | user-facing or product capability on a stack |
| `ops` | smoke, deploy, env activation, cron enablement |
| `docs` | docs-only |
| `unknown` | unclear |

### Explicit supersession

Parse body for:

- `## Supersedes` section (preferred; see `/issue` template)
- Phrases: “supersedes”, “replaces”, “instead of”, “do not use”, “abandoned”, “deferred in favor of”
- Issue ids `TEAM-\d+` named as replaced

### Stack verbs

`migrate to`, `remove`, `replace`, `canonical`, `system of record`, `not part of launch`, `superseded by`.

---

## Step C — Detect conflicts

| Type | Example |
|------|---------|
| Platform direction | One open ticket builds ClickHouse warehouse; another + docs say Neon only |
| Feature shape | Two open tickets redesign the same dashboard surface differently |
| Stale dependency | Feature AC requires ClickHouse tables while migration removes ClickHouse |
| Done vs open | Open ticket rewrites paths established by Done work (override, not skip Done) |

---

## Step D — Resolve canonical direction

Apply in order; record **confidence** (`high` | `medium` | `low`) and evidence:

1. Explicit `## Supersedes` / “canonical stack is X” on a **newer** open ticket.
2. **Repo docs + code** already on `main`/`dev` (e.g. “ClickHouse not part of launch”).
3. **Newer** issue (`createdAt` or higher number) with clear migration language vs older feature-on-old-stack.
4. If still ambiguous → **stop the batch**, show conflict table, ask the user. Do not invent a platform.

Outputs:

- `canonical_platforms: string[]`
- `abandoned_platforms: string[]`
- `direction_evidence: { issue_ids, doc_paths, notes }`

---

## Step E — Classify actions per leaf

| Situation | Action |
|-----------|--------|
| Open ticket **fully** targets abandoned platform; no remaining product value beyond that stack | `skip` + Linear comment; prefer **Canceled** or **Duplicate** of superseder when high confidence |
| Open feature targets abandoned platform but outcome still needed | `rescope` to canonical platform; rewrite AC in guidance/implement prompt |
| Open **migration** / foundation toward canonical | `promote` early in `order_rank` |
| Matches canonical; no conflict | `normal` |
| Partial supersession by newer open ticket | Older: implement only residual scope or skip if empty; newer: override rights |
| Supersedes **Done** work only | Implement superseder; document override of Done ids |

Never implement abandoned-platform work **before** the open migration that establishes the canonical platform when both are in the batch.

---

## Step F — Execution order

Assign `order_rank` (1 = first):

```text
1. Linear hard blockedBy unlockers (deps already modeled)
2. Architecture migrations / platform foundations for canonical direction
3. Re-scoped features that previously depended on abandoned stack
4. Independent features/docs (tie-break: lowest issue number)
5. Ops/smoke that need product code first — unless they gate all runtime
```

### Worked example

| Id | Intent | Outcome |
|----|--------|---------|
| LEET-67 | ClickHouse warehouse | skip / cancel (abandoned) |
| LEET-80 | Funnel rollups in ClickHouse | rescope → Neon rollups; after migration |
| LEET-123 | Edge → Neon ingestion | promote early |
| LEET-200 | Remove ClickHouse from launch | early/mid; may override Done ClickHouse artifacts |

Order sketch: `123` → `200` → re-scoped `80`; skip `67`.

---

## Step G — Write artifacts and report

1. Fill [`batch-guidance-template.md`](batch-guidance-template.md) → `guidance.md`.
2. Write `inventory.json`, `graph.json`, `state.json`.
3. User report (non-blocking unless Step D halted):

```text
Batch guidance: RUN_ID=… · path=<guidance.md>
Canonical: neon/postgres · Abandoned: clickhouse
Order: TEAM-123 → TEAM-200 → TEAM-80 (rescope)
Skip: TEAM-67 (full supersede / abandoned platform)
```

4. Proceed to claim/implement only non-skipped leaves in `order_rank` order (sequential) or wave order derived from the same ranks (fast).

---

## Refill (mid-batch)

After each successful Done (sequential) or merge wave (fast):

1. Re-list Linear for newly eligible leaves.
2. If new leaves add platform conflicts or supersession edges, **patch** guidance + re-rank **remaining** only.
3. Do not reorder already merged/solved work.
4. If `SELECTION_PIN` is set, do not append ids outside the pin ([`eligibility.md`](eligibility.md)).
5. If `SCOPE` is set, do not append ids outside `issue_in_scope` (same file, Scope filter).

---

## Linear policy (skips and rescopes)

| Event | Comment? | State change |
|-------|----------|--------------|
| Skip full-obsolete open ticket | **Yes** (superseded by …; batch guidance) | Prefer Canceled or Duplicate when high confidence; else leave open and skip implement |
| Rescope open ticket | **Yes** (re-scoped to canonical platform) | Leave actionable; implement re-scoped AC |
| Mere “not next in order” | **No** | Unchanged |
| Ambiguous direction stop | **No** mass cancels | Unchanged |

---

## Inject into implement prompts

Required block whenever S0 ran:

```markdown
## Batch guidance (hard)
- Path: <abs guidance.md>
- Canonical platforms: …
- Abandoned platforms: …
- This issue: order_rank=… · class=… · action=normal|rescope|…
- Supersedes: …
- Superseded by: …
- Override scope: …
- Re-scoped AC (if any): …
- Do **not** implement on abandoned platforms
- If guidance and the original ticket disagree on platform, **guidance wins**
```

---

## Anti-patterns

- Implementing ClickHouse/Convex (etc.) features before an open Neon migration when direction is Neon
- Treating lowest issue number as sacred across platform conflicts
- Reopening Done tickets instead of overriding via the newer ticket
- Canceling open tickets on low-confidence inference
- Letting implementers “honor the original AC” against guidance
- Skipping S0 on `/solve all` because fast mode “already has architecture.md”
