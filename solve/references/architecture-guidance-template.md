# Solve-Fast Architecture Guidance — {{PROJECT}} — {{RUN_ID}}

> **Note:** Prefer the full batch package from [`batch-guidance-template.md`](batch-guidance-template.md)
> as `guidance.md`. This template remains a **fast-mode supplement** for tech intersections and
> conflict zones. When both exist, **`guidance.md` wins** on platform direction and supersession.
>
> **Hard contract for every worker.** Read this entire document before coding.
> Do not invent parallel APIs, schemas, env keys, or naming that conflict with the shared contracts below.
> Do not implement abandoned platforms listed in batch guidance.
> Path on disk (orchestrator inlines absolute path into worker prompts): `{{ARCHITECTURE_MD_ABS}}`
> Machine graph: `{{GRAPH_JSON_ABS}}`

## Scope

| Field | Value |
|-------|--------|
| Run id | `{{RUN_ID}}` |
| Team / project | {{TEAM}} / {{PROJECT}} |
| Count mode | {{COUNT_MODE}} |
| Concurrency | {{CONCURRENCY}} |
| Effort | {{EFFORT}} |
| Repo root | {{REPO_ROOT}} |
| Local integration branch | `dev` (lowercase only) |
| Push / PR | **none** (default) |

### Eligible leaf inventory

{{ISSUE_INVENTORY}}

<!-- Example row:
- [TW-123](url) — title · wave 0 · deps: none · paths: apps/web/src/foo/**
-->

## Tech intersections

Technologies, packages, routes, schema areas, and auth surfaces that **two or more** issues in this run touch. Workers must align on these rather than inventing local alternatives.

{{TECH_INTERSECTIONS}}

<!-- Example:
- Neon schema / migrations under packages/db — issues TW-123, TW-140
- Auth session helpers in packages/auth — TW-124, TW-131
- Marketing SEO routes under apps/web/app/(marketing) — TW-150 only
-->

## Shared contracts (must follow)

Non-negotiable decisions for this run. If an issue needs something not listed, prefer existing repo patterns; do **not** introduce a second competing design for the same concern.

{{SHARED_CONTRACTS}}

<!-- Example:
1. API routes: continue existing `useRuntimeResource` / Neon runtime pattern; no new ad-hoc fetch wrappers.
2. DB: additive migrations only; no rename/drop of columns other tickets still use.
3. Env: only keys already in AGENTS.md / existing Doppler set; never commit secrets.
4. Naming: issue id prefix in commits (`TW-123: …`).
-->

## Explicit non-goals / do-not-touch

{{NON_GOALS}}

<!-- Example:
- Do not push remotes or open PRs.
- Do not rewrite the auth stack.
- Do not expand parent epic scope beyond this leaf issue.
-->

## Issue waves & merge order

Workers in the **same wave** may run in parallel (independent after hard deps + conflict serialization).
**Merge into local `dev` is always sequential** in `merge_order` within/across waves.
An issue **must not start** until every hard dependency is **merged to local `dev`** (not merely implemented in a worktree).

{{WAVES_AND_MERGE_ORDER}}

<!-- Example:
### Wave 0 (base = dev @ <sha>)
1. merge_order 1 — TW-123
2. merge_order 2 — TW-124

### Wave 1 (starts after wave 0 merges)
3. merge_order 3 — TW-125 (hard_deps: TW-123)
-->

## Per-issue notes

{{PER_ISSUE_NOTES}}

<!-- Example:
### TW-123 — title
- Parent epic: TW-100
- Hard deps: none
- Soft deps: none
- Primary paths: …
- Risks: …
- Acceptance highlights: …
-->

## Conflict zones (serialized)

Paths or modules that must **not** be edited by two concurrent workers. Issues listed under a zone run in **different waves** (or strict merge order with no overlap).

{{CONFLICT_ZONES}}

<!-- Example:
| Zone paths | Serialize (order) |
|------------|-------------------|
| packages/db/schema.ts | TW-123 → TW-140 |
-->

## Supersession / selective overrides

Later tickets may **replace** earlier functionality. When an issue is listed as a superseder, its AC wins inside the override scope; workers **must not** preserve superseded behavior “to stay compatible with TEAM-X.”

{{SUPERSESSION_EDGES}}

<!-- Example:
| Superseder | Mode | Superseded | Override scope | Notes |
|------------|------|------------|----------------|-------|
| TW-140 | full | TW-123 | entire prior surface for X | TW-123 skipped this run |
| TW-155 | partial | TW-130 (Done) | `apps/web/lib/foo.ts` | may rewrite foo |
-->

**Rules for workers:**

1. If your issue is a superseder: treat current AC as authoritative in the listed override scope; change/remove prior code as needed.
2. If your issue is **fully** superseded: you should not be spawned (orchestrator skips). If spawned in error, stop and report.
3. Shared contracts above must not force preservation of behavior that a superseder intentionally replaces — **superseder AC wins** for that scope.
4. Document overrides in the worker summary.

## Worker checklist

Before coding:

1. [ ] Read this file fully
2. [ ] Open your graph entry in `graph.json` for this issue id
3. [ ] Confirm hard deps are already on local `dev` (orchestrator should only spawn you when ready)
4. [ ] Check supersession edges: if you supersede others, plan selective override; if fully superseded, stop
5. [ ] Stay on the assigned issue branch in this worktree
6. [ ] Obey shared contracts **and** supersession override rules; smallest change meeting **current** acceptance criteria

After coding (worker):

1. [ ] Cheap construction (`solve-implementer`; bugs-only inner review on heavy/critical)
2. [ ] Verify per repo AGENTS / issue verification
3. [ ] Commit on issue branch only
4. [ ] Write worker summary to the path the orchestrator gave you
5. [ ] **Do not** merge `dev`, push, PR, or set Linear Done

---

## Orchestrator fill guide (not for workers)

When generating this document in Phase F2:

1. Inventory eligible **leaves** after epic expansion (same eligibility as sequential `/solve` Phase 2).
2. Pull Linear `blockedBy` / related → **hard_deps**.
3. Infer **primary_paths** from issue body, code map, labels, and light repo search.
4. Build **conflict_zones** where primary_paths overlap; serialize those issues.
5. Detect **supersedes** edges (`## Supersedes`, body language, related notes). Drop **full**-superseded leaves from the implement inventory; record edges in SUPERSESSION_EDGES and graph `supersedes` / `superseded_by` / `override_scope`.
6. For conflict zones involving a superseder + superseded: prefer serialize superseded → superseder only when the superseded issue is still being implemented (partial). Full supersedes: skip superseded.
7. Assign **waves** / **merge_order** (topo order; lowest issue number breaks ties within a level). Superseders of done work have no hard dep on the superseded id.
8. Extract **tech intersections** and **shared contracts** from the set of issues + repo conventions (`AGENTS.md`, existing patterns). Where a superseder replaces a contract, write the **new** contract and note the superseded id.
9. Keep the doc concise enough that every worker can read it in one pass; put bulk issue bodies in Linear, not here.
10. Write matching `graph.json` (see `graph.schema.json`) and keep both paths absolute in worker prompts.
