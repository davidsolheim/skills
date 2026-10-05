# Handoff Template (Phase 8 / D8)

Use this structure for the user-facing end summary. Keep it scannable. Full issue bodies live in `.wcp/issues/` (or in the local package if not filed).

---

## Filed successfully

```markdown
**Project review complete**
- **Mode:** <fast|deep>
- **Surface scope:** <whole project | …>
- **Repo / Notion:** origin URL · issues database
- **Live walk:** <yes URL | code-only | partial>
- **Filed:** yes

### Counts
| | P0 | P1 | P2 | Total |
|--|----|----|----|-------|
| Filed | n | n | n | n |
| Discovered not filed | n | n | n | n |
| Skipped duplicates | n | | | |

### Unblocked leaves (ready for `/solve`)
1. [0101](notion_url) — <title> · critical · foundation
2. [0102](notion_url) — <title> · high · feature
…

### Dependency chains
- 0103 blocked by 0102
- 0104 blocked by 0102

### Suggested next
```text
/solve
/solve 5
/solve all
/solve all
```
(Pick based on how hard you want to drain the queue.)

### Duplicates skipped
- Candidate “…” ≈ [0088](notion_url) — not re-filed (offline board_match)

### Retired
- [0040](notion_url) — contradicted filed leaf (canceled)

### Conflict — needs you
- [0055](notion_url) — in-progress / foreign claim; not canceled

### Drop-contradicted
- Candidate “…” vs [0080](notion_url) — user ticket is canonical; finding not filed

### Discovered not filed
- <title> (P2) — reason: fast mode filter | failed quality gate | …
- …

### Coverage notes
- <e.g. No staging URL; admin routes code-only>
- <fast: deferred deep a11y / secondary settings>

### Deep coverage (required when mode=deep)
- **Units:** <reviewed> reviewed / <pending> pending · <blocked_auth> blocked_auth · <out_of_scope> out_of_scope · <failed> failed
- **Slices / workers:** <N> slices · <failed workers> failed
- **Concurrency used:** <C>

### Scratch package
- **Status:** `deleted after successful file` **or** `kept` (only if not fully filed)
- **Path (if kept):** `<abs SCRATCH_DIR>` — required for re-file / draft
- **Path (if deleted):** n/a — all intended leaves are in `.wcp/issues/` above

When **kept** (draft / partial / not filed), also list:
- **Run:** `project-review-<RUN_ID>`
- **Guidance / inventory / coverage / board-snapshot / issue-candidates / index** under that path

### Assumptions
- <one-liners if product intent was inferred>
```

---

## Draft only / file write unfinished

```markdown
**Project review complete — NOT FILED**
- **Mode:** <fast|deep>
- **Reason:** `--draft` | file write unfinished: <brief>
- **Repo / Notion (intended):** origin URL

### Local package (**must keep** — not in `.wcp/issues/`)
- Scratch: `<abs path>` — **do not delete** until filed
- Finals ready: `…/issue-candidates/final/` (N files)
- Index: `…/issue-candidates/_merged/index.json`

### Draft package
(If no scratch: paste full draft package from notion-filing.md)

### Deep coverage
- Units: … (same breakdown as above)

### Suggested next
1. Publish from this scratch `final/` (no full rediscovery), then `/solve`
```

---

## Partial file failure

```markdown
**Project review partial file**
- Created: [list ids]
- Failed: [titles + error]
- **Scratch: KEPT** at `<abs path>` (do not delete — unfiled finals remain)
- Remaining finals on disk: `…/issue-candidates/final/` (list or count)
- Index updated for filed ids only

### Suggested next
- Re-file remaining finals from disk (do not re-run full deep discovery)
- `/solve` on successfully filed unblocked leaves
- After remaining finals are in `.wcp/issues/`, delete the scratch dir
```

---

## Tone rules

- Do not dump every checklist item examined
- Do not start implementing
- Do not auto-run `/solve` unless the user asked in the same turn
- One clear suggested command is better than five vague options
- In **fast** mode, always mention that **`/project-review deep`** remains available for a fuller pass
- In **deep** mode: show **coverage unit stats**; show **scratch path only when kept**; if fully filed, state scratch was **deleted after successful file**
- **Filed → delete temp. Not filed → keep temp.** Never imply recovery from a deleted package.
