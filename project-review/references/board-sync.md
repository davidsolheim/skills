# Board Sync — Snapshot, Offline Dedup, Relate

Goal: queue new work without flooding Linear with duplicates or fighting open platform direction — **without thrashing the Linear API**.

Deep mode: this is **Phase D1b (snapshot)** + **Phase D5 (offline match)**.  
Fast mode: one list/page of open work, then offline classification.

---

## Core rule

| Allowed | Forbidden |
|---------|-----------|
| **One** open-issue fetch (or a few pages) per run → write `board-snapshot.json` (deep: Phase D1b before workers; fast: before finalizing drafts) | Calling `list_issues` / search **per candidate** during discovery or cleanup |
| Offline title/path/keyword match against the snapshot file | Using Linear as the working set for draft tickets |
| Create pass at the end from cleaned local finals | Re-crawling the board mid-publish for routine dedupe |

Linear is for **snapshot keys** and **publish**, not iterative search.

---

## Steps

### 1. Snapshot open work (once)

Using Linear MCP (`list_issues`):

- Filter by resolved **team** and **project** when known
- Include open/actionable states (Backlog, Todo, In Progress, In Review, Blocked, Triage, …)
- Page until the open board for that project is reasonably complete (do not stop at one thin page if the board is large)
- Optionally sample **recent Done** titles for supersession context only

Capture into **`board-snapshot.json`** (and optional markdown index):

- identifier, title, state, labels, parent, blockedBy, url
- short description excerpt when available
- code-map paths if present in body

**Deep:** write under the review scratch dir (`$SCRATCH_DIR/board-snapshot.json`).  
**Fast:** same if using a scratch package; otherwise keep an in-memory/table snapshot for the rest of the run only — still do not re-fetch per candidate.

If Linear is unavailable: set snapshot empty/null; continue discovery; handoff notes offline-only dedupe.

### 2. Match each candidate (offline)

For each local candidate (files under `issue-candidates/` or candidate log), search the **snapshot only** by:

- Distinctive surface/route (`/billing`, “onboarding”)
- Error text or feature name
- Component/domain keywords from the code map
- Same epic theme (“empty state”, “a11y”, “projects list”)

Update candidate frontmatter / index:

- `board_match: TEAM-123` when high-confidence duplicate
- `related_board: [TEAM-…]` when related but not duplicate

### 3. Classify

| Classification | Criteria | Action |
|----------------|----------|--------|
| **Duplicate** | Same surface + same problem; existing ticket would be fixed by same AC | **Do not create.** `status: duplicate`, `board_match` set. Note in handoff. |
| **Related** | Same area, different problem | Keep for file; set `relatedTo` at publish using snapshot ids. |
| **Overlapping thin ticket** | Existing issue is vague; yours is solve-ready and same intent | Prefer **not** mass-rewriting others’ tickets. Create solve-ready leaf only if the thin ticket is clearly abandoned/unusable; else skip and list thin ticket in handoff as “needs rewrite via `/issue`”. Default: **skip create** when a reasonable open ticket already owns the problem. |
| **Conflict** | Open tickets redesign the same surface differently, or platform A vs B | Create only if your finding is still valid on canonical stack; fill `## Platform / stack` and `## Related`. Do not invent a migration ticket without repo evidence. |
| **New** | No meaningful overlap | Proceed to code pin + final/ + create. |

### 4. Platform / stack awareness

If snapshot issues or repo docs establish a platform direction (e.g. Neon vs abandoned Convex):

- Tag new leaves with `## Platform / stack`
- Avoid filing feature work that hard-requires an abandoned stack
- If a finding only makes sense post-migration, set `Migration dependency` and/or `blockedBy` when the migration issue exists in the snapshot

`/solve` batch guidance will further reorder/rescope; still avoid obvious obsolete-stack tickets.

### 5. Prior review epics

If an open epic already named like `Review pass (…)` for the same surface/month (in snapshot):

- Prefer adding children to **that** epic when still active and same intent, **or**
- Create a new dated epic if the prior pass is Done/Canceled or different surface

Do not create three parallel “Review pass” epics for the same deep audit without reason.

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

## Policy: do not spam Linear

- No comments on every skipped duplicate (handoff is enough)
- No mass status changes
- No canceling other people’s tickets from review
- No assigning yourself
- No mid-run board refresh loops (stale snapshot for one run is OK; unexpected create conflict → mark index and continue)

---

## Output

Update candidate index / log:

```text
CAND-003 → duplicate of TEAM-412 (skip create; board_match)
CAND-004 → new (keep → final/)
CAND-005 → related to TEAM-390 (keep + relatedTo at publish)
```

Only non-duplicate `ready_to_file` candidates enter Linear publish.

### Deep integration

| Phase | Board sync action |
|-------|-------------------|
| D1b | Write `board-snapshot.json` |
| D3–D4 | Workers do not call Linear |
| D5 | Offline match against snapshot |
| D7 | Publish; set relations from stored ids |
