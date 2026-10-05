# Board Sync — Snapshot, Offline Dedup, Relate

Goal: queue new work without flooding `.wcp/issues/` with duplicates **or leaving
contradicting directions both implementable**. Read the queue once into a snapshot.

Direction conflicts (classify + retire):
[`../../issue/references/direction-conflict.md`](../../issue/references/direction-conflict.md).
This file owns **when** to snapshot and that matching is **offline**. The
conflict procedure lives there.

Deep mode: this is **Phase D1b (snapshot)** + **Phase D5 (offline match)**.  
Fast mode: one read of `.wcp/issues/`, then offline classification.

---

## Core rule

| Allowed | Forbidden |
|---------|-----------|
| **One** read of `.wcp/issues/` per run → write `board-snapshot.json` (deep: Phase D1b before workers; fast: before finalizing drafts) | Re-listing the queue **per candidate** during discovery or cleanup |
| Offline title/path/keyword match against the snapshot file | Using Notion as the working set for draft tickets |
| Create pass at the end from cleaned local finals | Re-reading the board mid-publish for routine dedupe |

`.wcp/issues/` is the record. Snapshot once. Publish writes files, then Notion.

---

## Steps

### 1. Snapshot open work (once)

Read `.wcp/issues/` **once** (open, in-progress, blocked; skip done/canceled unless sampling recent done titles):

- Do not resolve a Linear team or project
- Capture id, title, status, priority, reason, files, notion_url
- short body excerpt
- code-map paths if present in body

Write **`board-snapshot.json`** (and optional markdown index). Workers do not re-list per candidate.

**Deep:** write under the review scratch dir (`$SCRATCH_DIR/board-snapshot.json`).  
**Fast:** same if using a scratch package; otherwise keep an in-memory/table snapshot for the rest of the run only — still do not re-read per candidate.

If the queue is empty or unreadable: set snapshot empty/null; continue discovery; handoff notes offline-only dedupe. Do not call Linear.

### 2. Match each candidate (offline)

For each local candidate (files under `issue-candidates/` or candidate log), search the **snapshot only** by:

- Distinctive surface/route (`/billing`, “onboarding”)
- Error text or feature name
- Component/domain keywords from the code map
- Same epic theme (“empty state”, “a11y”, “projects list”)

Update candidate frontmatter / index:

- `board_match: 0123` when high-confidence duplicate
- `related_board: [0123]` when related but not duplicate
- `contradicts_board: [0123]` + `retire_after_file: true` when this finding
  is canonical and unstarted board work is the abandoned direction
- `status: drop` / `drop-contradicted` when the finding loses to an explicit
  user ticket (repo does not pick the finding)

### 3. Classify

| Classification | Criteria | Action |
|----------------|----------|--------|
| **Duplicate** | Same surface + same problem; existing ticket would be fixed by same AC | **Do not create.** `status: duplicate`, `board_match` set. Note in handoff. |
| **Related** | Same area, different problem | Keep for file; note related ids in the leaf body. |
| **Overlapping thin ticket** | Existing issue is vague; yours is solve-ready and same intent | Prefer **not** mass-rewriting others’ tickets. Create solve-ready leaf only if the thin ticket is clearly abandoned/unusable; else skip and list thin ticket in handoff as “needs rewrite via `/issue`”. Default: **skip create** when a reasonable open ticket already owns the problem. |
| **Conflict** | Mutually exclusive intents on the same surface (or platform A vs B) | Follow direction-conflict.md **Authority (`/project-review`)**. If this finding is canonical: keep for file, `retire_after_file` unstarted ids, fill `## Supersedes` + Platform/stack. If an explicit user ticket is canonical: **do not create**; `drop-contradicted`. If ambiguous: needs-you; do not file; do not retire. Do not invent a migration ticket without repo evidence. |
| **New** | No meaningful overlap | Proceed to code pin + final/ + create. |

### 4. Platform / stack awareness

If snapshot issues or repo docs establish a platform direction (e.g. Neon vs abandoned Convex):

- Tag new leaves with `## Platform / stack`
- Avoid filing feature work that hard-requires an abandoned stack
- If a finding only makes sense post-migration, set `Migration dependency` and/or `reason: blocked by <id>` when the migration issue exists in the snapshot

Do **not** rely on `/solve` batch guidance to skip contradicted open tickets.
If this run files the canonical leaf, retire unstarted X at publish.

### 5. Prior review batches

If open leaves already carry an initiative line like `Review pass (…)` for the same surface/month (in snapshot):

- Prefer the same initiative name on new leaves when still active and same intent, **or**
- Use a new dated initiative line if the prior pass is done/canceled or a different surface

There is no epic file.

---

## Matching heuristics (practical)

Strong duplicate signals:

- Same route + same failure mode in title/body
- Same primary file path called out
- Existing AC already includes your expected outcome

Weak signals (usually **related**, not duplicate):

- Same page, different component
- Same design system theme, different instance
- “Polish dashboard” umbrella vs your atomic leaf — prefer atomic leaf **if** umbrella is too vague to implement; link related

---

## Policy: do not spam the queue

- No comments on every skipped duplicate (handoff is enough)
- No **mass** status changes
- Targeted **retire** of unstarted full contradictions at publish is required
  (direction-conflict.md). Live foreign claims / In Review: needs-you, no cancel.
- Do **not** retire a user ticket because the review merely invented the opposite
- No assigning yourself
- No mid-run board refresh loops (stale snapshot for one run is OK; unexpected create conflict → mark index and continue)

---

## Output

Update candidate index / log:

```text
CAND-003 → duplicate of 0412 (skip create; board_match)
CAND-004 → new (keep → final/)
CAND-005 → related to 0390 (keep; note related id in body)
CAND-006 → conflict; canonical finding; retire_after_file 0040
CAND-007 → drop-contradicted (user ticket 0080 is canonical)
```

Only non-duplicate, non-`drop-contradicted` `ready_to_file` candidates enter the publish pass.

### Deep integration

| Phase | Board sync action |
|-------|-------------------|
| D1b | Write `board-snapshot.json` |
| D3–D4 | Workers do not call Linear |
| D5 | Offline match against snapshot |
| D7 | Publish; set relations; **retire** `retire_after_file` ids after create |
