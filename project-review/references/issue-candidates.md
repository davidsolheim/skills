# Issue candidates — local package (file-first)

Canonical layout and lifecycle for **local** issue candidates under `/project-review`.

Parent: [`deep-mode.md`](deep-mode.md) · Filing: [`linear-filing.md`](linear-filing.md) · Board: [`board-sync.md`](board-sync.md)

---

## Principle

```text
Workers invent → write local candidate files
     ↓
Orchestrator merges / dedupes / quality-gates on disk
     ↓
--draft stops here  OR  single Linear publish from final/
```

Linear is the **publish** step, not the working set.

- **Do not** thrash Linear’s API while inventing or merging candidates.
- **Do not** create tickets from raw worker dumps.
- Deduplication uses the **board snapshot file** + local file comparison.

Fast mode should still prefer a small local candidates folder before create; deep **requires** this tree.

---

## Directory layout

Under `$SCRATCH_DIR/issue-candidates/`:

```text
issue-candidates/
  _inbox/
    <slice_id>/
      CAND-001-empty-state.md
  by-route/
    dashboard/
      CAND-001-empty-state.md
    settings-billing/
      CAND-022-validation.md
  by-api/
    projects-create/
      CAND-031-error-handling.md
  by-component/
    data-table/
      CAND-040-loading-skeleton.md
  cross-cutting/
    CAND-050-button-variant-drift.md
  _merged/
    index.json
  final/
    CAND-001-empty-state.md
    CAND-022-validation.md
```

| Path | Role |
|------|------|
| `_inbox/<slice_id>/` | Append-only raw worker output (first write) |
| `by-route/<route-slug>/` | Primary surface is a route/view |
| `by-api/<api-slug>/` | Primary surface is API/server action |
| `by-component/<name>/` | Shared UI / design-system finding |
| `cross-cutting/` | App-wide consistency, a11y programs, content voice |
| `_merged/index.json` | Machine index over all candidates |
| `final/` | **Publish set only** — full issue-template bodies |

### Slug rules

- Route slug: strip leading `/`, replace `/` with `-` (`/settings/billing` → `settings-billing`).
- API slug: method + path or action name (`projects-create`, `api-users-id`).
- Keep names filesystem-safe, lowercase, short.

### One finding per file

Preferred. Enables parallel writes, easy drop/duplicate, and clear final publish.

---

## Frontmatter schema

```yaml
---
id: CAND-014
status: draft | keep | drop | duplicate | ready_to_file | filed
priority: P0 | P1 | P2
lens: Completeness | Functional | Edge | UI | Hierarchy | Taste | Responsiveness | A11y | Performance | Content | Cross-feature
surface: /dashboard
unit_ids: [apps-web-route-dashboard]
primary_paths: [apps/web/app/dashboard/page.tsx]
duplicate_of: null
board_match: null
related_board: []
class: foundation | feature | polish | content | a11y
blocked_by_candidates: []
linear_id: null
linear_url: null
slice_id: web-dashboard
---
```

| Field | Owner |
|-------|--------|
| id, thin body | Worker |
| status transitions, duplicate_of, board_match | Orchestrator (D5) |
| full template sections | Orchestrator / pin workers |
| linear_id / linear_url | Orchestrator (D7) |

---

## Worker write rules

1. Always write a copy (or primary) under `_inbox/<slice_id>/`.
2. Also place under `by-route|by-api|by-component|cross-cutting` when primary surface is known (hard link, copy, or single path recorded in index — prefer **one canonical path** + index entry; if tools make hard links hard, write once to the best by-* folder **and** list path in worker report; orchestrator may copy into `_inbox` index on ingest).
3. **Practical default for workers:** write the file to the best `by-*` or `cross-cutting` path **and** ensure `_inbox/<slice_id>/` contains the same file (copy is fine).
4. Thin drafts OK: title, current/expected, evidence, paths, priority, lens.
5. Never write under `final/` (orchestrator only).

---

## Lifecycle

| Stage | Status values | Location |
|-------|---------------|----------|
| Invented | `draft` | `_inbox` + `by-*` |
| Kept after signal filter | `keep` | same |
| Near-dupe or board dupe | `duplicate` | same; `duplicate_of` or `board_match` set |
| Nit / weak | `drop` | same; excluded from final |
| Full template ready | `ready_to_file` | **copied/rendered to `final/`** |
| Linear created | `filed` | index updated with `linear_id`; after **full** publish verify, entire scratch dir may be **deleted** (see lifecycle below) |

---

## Phase D5 — Cleanup procedure

### 1. Ingest

Scan `_inbox/**/*.md` and `by-*/**/*.md` + `cross-cutting/**/*.md`.  
Build/update `_merged/index.json`:

```json
{
  "run_id": "...",
  "candidates": [
    {
      "id": "CAND-014",
      "path": "issue-candidates/by-route/dashboard/CAND-014-empty-state.md",
      "status": "draft",
      "priority": "P1",
      "surface": "/dashboard",
      "primary_paths": ["apps/web/app/dashboard/page.tsx"],
      "duplicate_of": null,
      "board_match": null,
      "related_board": [],
      "blocked_by_candidates": [],
      "linear_id": null,
      "title": "Fix empty state on /dashboard"
    }
  ],
  "stats": {
    "draft": 0,
    "keep": 0,
    "drop": 0,
    "duplicate": 0,
    "ready_to_file": 0,
    "filed": 0
  }
}
```

### 2. Near-dupe merge (local only)

Merge when:

- Same `surface` (or overlapping primary_paths) **and** same root-cause problem, or
- Titles/AC would produce the same implementable change

Keep the strongest write-up (clearest AC + best pin). Set others `status: duplicate`, `duplicate_of: CAND-…`.

### 3. Board match (offline only)

Against `board-snapshot.json` **only** — do **not** re-call `list_issues`.

| Confidence | Action |
|------------|--------|
| High same surface + same problem | `status: duplicate`, `board_match: TEAM-123` — **not** in final/ |
| Same area, different problem | `related_board: [TEAM-123]`; keep for file; later `relatedTo` |
| Vague open umbrella vs atomic leaf | Prefer keep atomic if umbrella unimplementable; else skip if umbrella owns it; note in handoff |

### 4. Signal filter

Promote/drop per [`review-checklist.md`](review-checklist.md) filter.  
Deep: keep well-formed P2; drop pure nits.

### 5. Code pin + taste

For each `keep`:

- Fill code map + drift check
- Run taste-to-concrete when needed
- May spawn read-only pin workers over batches of candidate paths

If still unpinnable: drop or demote with explicit Assumptions — never ship zero-anchor polish to final/.

### 6. Render final/

For each `ready_to_file`, write full [`issue-template.md`](issue-template.md) body to:

```text
issue-candidates/final/CAND-014-empty-state.md
```

Include frontmatter + full sections. Filename stable with candidate id.

### 7. Quality gate

Every `final/*.md` must pass the Phase 5 checklist in SKILL.md. Failures leave status `keep` (not ready) until fixed or dropped.

### 8. Dependency plan

Set `blocked_by_candidates` on index entries (candidate ids). Used in D6/D7 after Linear ids exist.

---

## Phase D7 — Publish

1. Walk `final/*.md` only (filter by filing flags: P0/P1 only if `--p0-p1-only`).
2. Create Linear issues; set `linear_id`, `linear_url`, `status: filed` in index.
3. Apply relations from `related_board` and `blocked_by_candidates` (map candidate → linear_id).
4. **Scratch lifecycle** (see below).

On failure: remaining `final/` files stay on disk for re-file without re-running discovery — **do not delete** scratch.

---

## Scratch directory lifecycle

The whole `$SCRATCH_DIR` (`…/project-review-<RUN_ID>/`), not only `issue-candidates/`:

| Condition | Action |
|-----------|--------|
| Intended publish set is **in Linear** (every published final has `linear_id`; verified) | **Delete** `$SCRATCH_DIR` |
| Not in Linear (`--draft`, auth fail, partial file, never published) | **Keep** `$SCRATCH_DIR` |

**Filed → delete. Not filed → keep.**

Record Linear ids/URLs in the handoff **before** any delete. Do not delete the parent `grok-$(id -u)/` directory.

---

## `--draft` behavior

Stop after D5/D6. Handoff points at:

- `issue-candidates/final/` (ready bodies)
- `_merged/index.json`
- Drops/duplicates summary
- Absolute scratch path (**must keep** — nothing in Linear yet)

No Linear create. **Do not delete** scratch.

---

## Fast mode (lightweight)

May use:

```text
$SCRATCH_DIR/issue-candidates/final/
$SCRATCH_DIR/issue-candidates/_merged/index.json
```

without full by-route fan-out. Still: **clean locally → then file**. Prefer one board snapshot over per-ticket Linear search. Same scratch rule: delete after verified full Linear file; keep if draft/partial.

---

## Anti-patterns

- Filing Linear from `_inbox` or uncleaned drafts
- Calling Linear list/search for each candidate during cleanup
- One giant `candidates.md` that workers fight over
- Deleting raw inbox before index records paths (keep until run completes)
- Publishing without board_match pass when snapshot exists
- Equating “unit reviewed” with “must emit a candidate”
- **Deleting `$SCRATCH_DIR` while any intended `final/` is unfiled**
- **Leaving `$SCRATCH_DIR` after a fully verified Linear publish** (delete the run dir)
