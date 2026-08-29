# Solve batch guidance — {{PROJECT}} — {{RUN_ID}}

> **Hard contract for every implementer in this batch.** Read this entire document before coding.
> When this file and an older Linear ticket disagree on **platform / stack / architectural direction**, **this file wins**.
> Path on disk: `{{GUIDANCE_MD_ABS}}`
> Graph: `{{GRAPH_JSON_ABS}}` · Inventory: `{{INVENTORY_JSON_ABS}}`

## Run metadata

| Field | Value |
|-------|--------|
| Run id | `{{RUN_ID}}` |
| Mode | {{MODE}} <!-- sequential \| fast --> |
| Team / project | {{TEAM}} / {{PROJECT}} |
| Count mode | {{COUNT_MODE}} |
| Effort | {{EFFORT}} |
| Repo root | {{REPO_ROOT}} |
| Integration branch | `dev` (lowercase) |
| Push / PR | **none** (default) |

## Repo architecture truth

What the **codebase and docs on main/dev** say today (not what old tickets hoped).

{{REPO_TRUTH}}

<!-- Example:
- Operational DB: Neon Postgres only (`apps/web`, `apps/neon-ingest`).
- ClickHouse warehouse plan: superseded / not launch (`docs/plans/…`, `docs/architecture.md`).
- Convex: not day-one; only if realtime need is proven.
-->

## Canonical direction (this batch)

| Field | Value |
|-------|--------|
| Canonical platforms | {{CANONICAL_PLATFORMS}} |
| Abandoned platforms | {{ABANDONED_PLATFORMS}} |
| Confidence | {{DIRECTION_CONFIDENCE}} <!-- high \| medium \| low --> |
| Evidence | {{DIRECTION_EVIDENCE}} |

If confidence is **low**, the orchestrator must **stop and ask the user** before claiming work.

## Eligible inventory

{{ISSUE_INVENTORY}}

<!-- Example row:
- [TW-123](url) — title · class=foundation · platforms=[neon] · action=normal · paths: apps/…
-->

## Direction conflicts

{{DIRECTION_CONFLICTS}}

<!-- Example:
- TW-67 (clickhouse feature) vs TW-200 (remove clickhouse) + repo truth Neon-only
- TW-80 assumes ClickHouse rollups; TW-123 establishes Neon ingest
-->

## Supersession edges

| Superseder | Mode | Superseded | Override scope | Notes |
|------------|------|------------|----------------|-------|
{{SUPERSESSION_TABLE}}

<!-- mode: full | partial -->

**Rules:**

1. Superseder AC is authoritative inside override scope.
2. Implementers **may change/delete** code that satisfied superseded tickets inside that scope.
3. Fully superseded **open** tickets are **not implemented as written** (see skip table).
4. **Done** tickets are not reopened; superseders override their artifacts in code only.

## Skip / cancel / re-scope

| Issue | Action | Reason | Linear note |
|-------|--------|--------|-------------|
{{SKIP_RESCOPE_TABLE}}

<!-- action: skip | cancel | duplicate | rescope -->

## Execution order

Implement **only** non-skipped issues, in this order (unless hard_deps force a wait).  
Lowest issue number is **only** a tie-breaker among independent peers.

{{EXECUTION_ORDER}}

<!-- Example:
1. TW-123 — foundation/migration (Neon ingest) · why: canonical platform foundation
2. TW-200 — migration cleanup · why: abandons ClickHouse
3. TW-80 — feature (rescope to Neon) · why: after migration; do not build ClickHouse
SKIP: TW-67 — abandoned platform
-->

## Per-issue implement notes

{{PER_ISSUE_NOTES}}

<!-- Example:
### TW-80 — funnel rollups
- action: rescope
- original stack: clickhouse → implement on: neon/postgres rollups
- supersedes: none
- do not: add ClickHouse client, warehouse tables, or dual-write
- override Done: n/a
-->

## Shared contracts (must follow)

Non-negotiable for this run. Prefer existing repo patterns. Do **not** reintroduce abandoned platforms.

{{SHARED_CONTRACTS}}

## Conflict zones (path serialization)

Paths that must not be edited by two concurrent workers (fast mode). Sequential mode still uses this as a merge caution list.

{{CONFLICT_ZONES}}

## Ambiguities / human gates

{{AMBIGUITIES}}

<!-- Empty if none. If non-empty and blocking, orchestrator stops. -->

## Worker / implementer checklist

1. [ ] Read this file fully  
2. [ ] Confirm this issue is **not** in the skip table  
3. [ ] Obey canonical / abandoned platforms  
4. [ ] Apply re-scope notes if present (ignore obsolete stack AC)  
5. [ ] Apply selective override only inside listed scope  
6. [ ] Smallest change meeting **current** (possibly re-scoped) AC  
7. [ ] Summary lists supersession/rescope compliance  
8. [ ] Do **not** merge `dev`, push, PR, or set Linear Done (orchestrator owns those)

---

## Orchestrator fill guide

1. Run full inventory (eligible leaves + repo truth).  
2. Tag platforms, class, explicit supersedes.  
3. Resolve canonical vs abandoned direction (see `batch-guidance.md`).  
4. Build supersession + skip/rescope tables.  
5. Assign `order_rank` (migration/foundation before re-scoped features).  
6. Write graph.json / inventory.json / state.json.  
7. Report order + skips to the user; halt only on low-confidence direction conflicts.  
8. Inject this file’s absolute path into every implement prompt.  
9. After each Done/merge, refill and patch remaining order if needed.
