# Publish pass

**Do not call Linear.** Write each ready leaf into `.WCP/issues/` per [`../../docs/wcp-queue.md`](../../docs/wcp-queue.md), then upsert Notion per [`../../docs/notion-issues.md`](../../docs/notion-issues.md). Independent leaves go in `open/` at Status `open`. A hard dependency goes in `blocked/` with `reason` and Notion Status `blocked`. There is no epic file. `--draft` keeps the local package and writes nothing.

Default: **file** solve-ready leaves so `/solve` has a real queue.

**Deep:** this is Phase D7 — publish **only** from cleaned local files.  
**Fast:** same idea when using `issue-candidates/final/`; otherwise create from equivalent full drafts after cleanup.

---

## Preconditions

- Notion database for this repo ([`../../docs/notion-issues.md`](../../docs/notion-issues.md))
- Board **snapshot** already taken — offline match done ([`board-sync.md`](board-sync.md))
- Every leaf to file passed quality gates
- Dependency order planned
- **Deep:** candidates live in `issue-candidates/final/*.md` with `status: ready_to_file` (or equivalent)
- **Never** create from raw `_inbox/` or uncleaned worker dumps

---

## Source of truth

| Mode | Create from |
|------|-------------|
| Deep | `$SCRATCH_DIR/issue-candidates/final/*.md` only |
| Fast | `final/*.md` if present; else full in-memory/draft bodies that passed gates |
| Draft | Nothing filed; keep `final/` on disk |

After create, write the issue id into `_merged/index.json` when that index exists.

---

## Scratch cleanup after publish

| Result | Action |
|--------|--------|
| Publish **fully succeeded** (every intended final has an issue id) | **Delete** `$SCRATCH_DIR` (`rm -rf` the `project-review-<RUN_ID>` dir only) |
| `--draft`, Notion unavailable, or **any** intended final not filed | **Keep** `$SCRATCH_DIR` — required for re-file without rediscovery |

Capture leaf ids for the handoff **before** deleting.  
If deleted, handoff says scratch was removed after successful file (no path needed for recovery).  
If kept, handoff **must** include the absolute scratch path.

Filtered-out discoveries (signal filter / `--p0-p1-only` never in `final/`) do not force keep. Unfiled bodies still in `final/` **do** force keep.

---

## Notion

After each file write, upsert that row ([`../../docs/notion-issues.md`](../../docs/notion-issues.md)). Never put secrets in titles, bodies, or Notion properties. Do not re-read the whole queue per leaf. On a duplicate file, skip the create and continue.

---

## Filing filters

| Condition | File |
|-----------|------|
| `FILE_MODE=draft` | Nothing |
| Fast default | P0 + P1 only |
| Fast + `--include-p2` | P0 + P1 + high-leverage P2 that passed gates |
| Deep default | All well-formed P0/P1/P2 in `final/` |
| Deep + `--p0-p1-only` | P0 + P1 only from `final/` |

Unfiled discoveries go in handoff under **Discovered not filed** (and remain on disk under `by-*` / index as `drop` / filtered).

---

## Priority map

| Review | priority |
|--------|----------|
| P0 data loss / broken core production path | `critical` |
| Other P0 / most high-visibility P1 | `high` |
| Remaining P1 / solid P2 | `normal` |
| Minor P2 | `low` |

---

## State

New leaves are `open` and unassigned. Do not set `in-progress` or `done`. `/solve` claims the file. Notion `done` waits for `/prb` or `/yeet`.

## Create sequence

1. For each ready final, in dependency order, write the queue file and upsert Notion.
2. A hard dependency is `blocked/` with `reason: blocked by <id>`.
3. Put the initiative name in the leaf body. Do not write an epic file.
4. Record the new id in the local index.
5. Retire contradicted unstarted files per [`../../issue/references/direction-conflict.md`](../../issue/references/direction-conflict.md). Skip a file under a live lease. `--draft` writes nothing.

---

## Failure handling

| Failure | Action |
|---------|--------|
| Notion auth error | Stop filing; **keep** full scratch package; handoff **not filed** with absolute path |
| Database mismatch | Fix the slug or origin URL; retry once |
| Single leaf fails | Continue others; report failed path; **keep** scratch (partial) |
| Blocked `reason` missing | Leaves still valid; note the missing reason in the handoff; **keep** scratch if the dependency plan was material |
| Unexpected duplicate on create | Mark index; do not re-read the whole queue; continue |

Never pretend issues were created when they were not.  
Never re-run full deep discovery solely because publish failed — re-file from `final/` while scratch is kept.  
Never delete scratch until publish is fully verified.

---

## Draft package format (when not filing)

Prefer pointing at disk:

```markdown
# Project review drafts — NOT FILED

Mode: deep
Repo: …
Scratch: /tmp/grok-…/project-review-<RUN_ID>/
Finals: …/issue-candidates/final/
Index: …/issue-candidates/_merged/index.json

## Finals ready to publish
1. CAND-001 — <title> (P0, foundation) — path: …
2. CAND-002 — …
```

If no scratch dir (legacy fast), emit pasted bodies:

```markdown
# Project review drafts — NOT FILED

Mode: fast
…

## 1. <title> (P0, foundation)
<full body>
```

---

## What this skill must never do

- Assign to self or others
- Set `in-progress` or `done` on **new** leaves
- Set Notion `done` (`/prb` or `/yeet` owns that)
- Mass-cancel pre-existing issues
- Skip targeted retire of unstarted full contradictions (`retire_after_file`)
- Write an epic file
- Create from uncleaned `_inbox` dumps
- Per-candidate queue search during publish

---

## Alignment with `/solve`

| Moment | Review | Solve |
|--------|--------|-------|
| Create `open/` leaves and Notion rows | Yes (publish pass) | No |
| Claim / `in-progress` | No | Yes, and Notion `in-progress` |
| Set the local file `done` | No | Reviewer, after the check |
| Set Notion `done` | No | No (`/prb` or `/yeet`) |
