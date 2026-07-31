# Project review guidance — {{PROJECT}} — {{RUN_ID}}

> **Hard contract for every review worker in this run.** Read this entire document before reviewing.
> Write findings **only** as local files under `issue-candidates/`. Do **not** call Linear, create tickets, implement fixes, or edit application source.
>
> Path on disk: `{{GUIDANCE_MD_ABS}}`  
> Scratch: `{{SCRATCH_DIR_ABS}}`  
> Inventory: `{{INVENTORY_JSON_ABS}}` · Coverage: `{{COVERAGE_JSON_ABS}}`  
> Candidates: `{{ISSUE_CANDIDATES_DIR_ABS}}` · Board snapshot: `{{BOARD_SNAPSHOT_ABS}}`

## Run metadata

| Field | Value |
|-------|--------|
| Run id | `{{RUN_ID}}` |
| Mode | deep |
| Team / project | {{TEAM}} / {{PROJECT}} |
| Concurrency | {{CONCURRENCY}} |
| Repo root | {{REPO_ROOT}} |
| Live URL | {{LIVE_URL}} |
| Scope only | {{SCOPE_ONLY}} |
| File mode | {{FILE_MODE}} <!-- file \| draft --> |
| Surface bias | {{SURFACE_BIAS}} |

## Intended state (agent-built)

{{INTENDED_STATE}}

<!-- Include: primary_journeys, secondary_features, constraints, non-goals, confidence, assumptions -->

## Good reference surfaces (match these)

{{REFERENCE_SURFACES}}

<!-- e.g. /settings billing cards; marketing landing hero; empty state on /projects when good -->

## Repo truth (stack / packages)

{{REPO_TRUTH}}

## Quality bar (non-negotiable)

Every candidate that may become a Linear leaf must eventually support:

1. Atomic scope (one primary change)
2. Current behavior + expected behavior
3. Checklist acceptance criteria
4. Code map + drift check (pin phase may deepen)
5. Verification commands/steps
6. Out of scope / do not change
7. Taste fully concretized when visual/feel (match surface X or exact tokens — no “make it premium”)
8. No secrets

Authoritative templates (orchestrator expands thin worker drafts):

- Issue body: skill `references/issue-template.md`
- Taste conversion: skill `references/taste-to-concrete.md`
- Signal filter: skill `references/review-checklist.md` (bottom)

## Lenses required (deep)

For **each** assigned unit, apply checklist items tagged `[F]` and `[D]`:

Completeness · Functional · Edge/empty/error · UI consistency · Hierarchy · Interaction/taste · Responsiveness · A11y · Perceived performance · Content · Cross-feature · Auth · Nav/stubs · Intended-but-stubbed code signals

Do **not** create one ticket per checkbox. Checklist is a search pattern. Empty findings for a clean unit are valid.

## Inventory summary

{{INVENTORY_SUMMARY}}

<!-- counts by kind; total units; primary vs secondary -->

## Your slice

| Field | Value |
|-------|--------|
| Slice id | *(filled per worker)* |
| Unit ids | *(filled per worker)* |
| Package focus | *(filled per worker)* |

## Where to write candidates

| Stage | Path |
|-------|------|
| Raw worker dumps | `issue-candidates/_inbox/<slice_id>/CAND-###-slug.md` |
| Route-primary | `issue-candidates/by-route/<route-slug>/…` |
| API-primary | `issue-candidates/by-api/<api-slug>/…` |
| Component-primary | `issue-candidates/by-component/<name>/…` |
| Cross-cutting | `issue-candidates/cross-cutting/…` |

**Filename:** `CAND-<nnn>-<short-slug>.md` (unique nnn within the run; prefer slice prefix if coordinating: `CAND-<slice>-001-…` is OK if globally unique).

### Minimum frontmatter

```yaml
---
id: CAND-014
status: draft
priority: P1
lens: Edge
surface: /dashboard
unit_ids: [apps-web-route-dashboard]
primary_paths: [apps/web/app/dashboard/page.tsx]
duplicate_of: null
board_match: null
class: feature
---
```

### Minimum body (worker may be thin; orchestrator expands)

- Working title (H1)
- Current behavior + evidence
- Expected behavior sketch
- Code paths if known
- Priority + lens

## Coverage report (required)

Write `workers/<slice_id>.md`:

```markdown
## Slice <id>
### Units reviewed
- unit_id · kind · paths · lenses_run · findings_count · status · notes
### Candidates written
- path list
### Gaps / blocked
- none | list
```

Mark each unit terminal in spirit: reviewed with zero findings is success. Note `blocked_auth` when live path unreachable but code review done.

## Verification commands (for later ticket bodies)

{{VERIFICATION_COMMANDS}}

## Board snapshot (workers: read-only if needed)

Workers **do not** call Linear. Optional: skim `board-snapshot.json` only to avoid inventing obvious duplicates already titled on the board. Prefer writing the candidate and letting orchestrator set `board_match` offline.

## Forbidden

- [ ] Linear MCP / create issue / assign / comment
- [ ] Edit application source, config, or tests
- [ ] `git commit` / push / PR
- [ ] Implement fixes
- [ ] Claim the run is complete or tickets are filed
- [ ] Stop early with assigned units unreviewed

## Worker checklist

1. [ ] Read this file fully  
2. [ ] Review every assigned unit (full deep lenses)  
3. [ ] Write candidates under issue-candidates/ only  
4. [ ] Write workers/<slice_id>.md  
5. [ ] Zero findings units still listed as reviewed  
6. [ ] No Linear, no app edits  

---

## Orchestrator fill guide

1. Complete Phase D0 inventory.  
2. Fill this template → `guidance.md`.  
3. Init coverage.json + issue-candidates tree.  
4. Phase D1b board snapshot.  
5. Per worker: inject slice id + unit list + absolute paths.  
6. After all units terminal → cleanup → final/ → Linear publish from final/ only.
