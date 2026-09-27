# Board Sync — Snapshot, Offline Dedup, Relate

Goal: queue new work without filing duplicates or leaving contradicting directions both implementable.

The board is `.wcp/issues/` ([`../../docs/wcp-queue.md`](../../docs/wcp-queue.md)). Notion status is [`../../docs/notion-issues.md`](../../docs/notion-issues.md). Do not call Linear or GitHub Issues.

Direction conflicts (classify + retire):
[`../../issue/references/direction-conflict.md`](../../issue/references/direction-conflict.md).
This file owns **when** to snapshot and that matching is **offline**. The
conflict procedure lives there.

Deep mode: this is **Phase D1b (snapshot)** + **Phase D5 (offline match)**.
Fast mode: one read of the open queue, then offline classification.

---

## Core rule

| Allowed | Forbidden |
|---------|-----------|
| **One** read of `open/`, `in-progress/`, and `blocked/` per run → write `board-snapshot.json` (deep: Phase D1b before workers; fast: before finalizing drafts) | Re-reading the queue **per candidate** during discovery or cleanup |
| Offline title/path/keyword match against the snapshot file | Using Notion as the working set for draft tickets |
| Write `.wcp/issues/` and upsert Notion at the end from cleaned local finals | Searching the queue mid-publish for routine dedupe |

The queue files are for **snapshot keys** and **publish**, not iterative search.

---

## Steps

### 1. Snapshot open work (once)

Read every `*.md` under:

- `.wcp/issues/open/`
- `.wcp/issues/in-progress/`
- `.wcp/issues/blocked/`

Skip `done/` and `canceled/` for the working set. Optionally sample recent `done/` titles for supersession context only.

Capture into **`board-snapshot.json`** (and an optional markdown index):

- `id`, title, `status`, folder, `priority`
- `scope`, `acceptance`, `reason` when present
- code-map paths if present in the body

**Deep:** write under the review scratch dir (`$SCRATCH_DIR/board-snapshot.json`).
**Fast:** same if using a scratch package; otherwise keep an in-memory snapshot for the rest of the run only — still do not re-read per candidate.

If `.wcp/issues/` is missing: set the snapshot empty and continue discovery. Say that dedupe was local-only.

### 2. Match each candidate (offline)

For each local candidate (files under `issue-candidates/` or the candidate log), search the **snapshot only** by:

- Distinctive surface/route (`/billing`, “onboarding”)
- Error text or feature name
- Component/domain keywords from the code map
- Same theme (“empty state”, “a11y”, “projects list”)

Update candidate frontmatter / index:

- `board_match: 0123` when high-confidence duplicate
- `related_board: [0123]` when related but not duplicate
- `contradicts_board: [0123]` + `retire_after_file: true` when this finding
  is canonical and the unstarted queue file is the abandoned direction
- `status: drop` / `drop-contradicted` when the finding loses to an explicit
  user ticket (the review does not pick the finding)

### 3. Classify

| Classification | Criteria | Action |
|----------------|----------|--------|
| **Duplicate** | Same surface + same problem; existing file would be fixed by the same acceptance | **Do not create.** `status: duplicate`, `board_match` set. Note in handoff. |
| **Related** | Same area, different problem | Keep for file; name the related id in the leaf body. |
| **Overlapping thin ticket** | Existing file is vague; yours is solve-ready and the same intent | Prefer **not** mass-rewriting others’ tickets. Create a solve-ready leaf only if the thin file is clearly abandoned; else skip and list it in the handoff as “needs rewrite via `/issue`”. Default: **skip create** when a reasonable open file already owns the problem. |
| **Conflict** | Mutually exclusive intents on the same surface (or platform A vs B) | Follow direction-conflict.md **Authority (`/project-review`)**. If this finding is canonical: keep for file, `retire_after_file` unstarted ids, fill `## Supersedes`. If an explicit user ticket is canonical: **do not create**; `drop-contradicted`. If ambiguous: needs-you; do not file; do not retire. |
| **New** | No meaningful overlap | Proceed to code pin + final/ + create. |

### 4. Platform / stack awareness

If snapshot files or repo docs establish a platform direction (for example Neon vs an abandoned stack):

- Tag new leaves with `## Platform / stack`
- Avoid filing feature work that hard-requires an abandoned stack
- If a finding only makes sense after a migration, set `Migration dependency` and/or `blocked by <id>` when that migration file exists in the snapshot

Do **not** rely on `/solve` batch guidance to skip contradicted open tickets.
If this run files the canonical leaf, retire unstarted contradicted files at publish.

### 5. Prior review batches

If an open file already names a review pass for the same surface:

- Prefer not filing a second leaf with the same acceptance
- A new dated batch is fine when the prior pass is `done` or `canceled`, or the surface differs

---

## Matching heuristics (practical)

Strong duplicate signals:

- Same route + same failure mode in title/body
- Same primary file path called out
- Existing acceptance already includes the expected outcome

Weak signals (usually **related**, not duplicate):

- Same page, different component
- Same design-system theme, different instance
- A vague umbrella vs an atomic leaf — prefer the atomic leaf **if** the umbrella is too vague to implement, and name the related id

---

## Policy: do not spam the queue

- No Notion writes on every skipped duplicate (handoff is enough)
- No **mass** status changes
- Targeted **retire** of unstarted full contradictions at publish is required
  (direction-conflict.md). A live lease: needs-you, no cancel.
- Do **not** retire a user ticket because the review merely invented the opposite
- No assigning
- No mid-run queue refresh loops (a stale snapshot for one run is OK)

---

## Output

Update the candidate index / log:

```text
CAND-003 → duplicate of 0042 (skip create; board_match)
CAND-004 → new (keep → final/)
CAND-005 → related to 0039 (keep; name 0039 in the body)
CAND-006 → conflict; canonical finding; retire_after_file 0040
CAND-007 → drop-contradicted (user ticket 0080 is canonical)
```

Only non-duplicate, non-`drop-contradicted` `ready_to_file` candidates are written into `.wcp/issues/`.

### Deep integration

| Phase | Board sync action |
|-------|-------------------|
| D1b | Write `board-snapshot.json` from `.wcp/issues/` |
| D3–D4 | Workers do not read the queue and do not call Notion |
| D5 | Offline match against the snapshot |
| D7 | Publish; **retire** `retire_after_file` ids after the new files exist |
