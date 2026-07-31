# Linear Filing — Publish Pass & Policy

Default: **file** solve-ready leaves so `/solve` has a real queue.  
`--draft` or Linear failure → local markdown package only.

**Deep:** this is Phase D7 — publish **only** from cleaned local files.  
**Fast:** same idea when using `issue-candidates/final/`; otherwise create from equivalent full drafts after cleanup.

---

## Preconditions

- Team resolved (required)
- Project resolved when identifiable
- Board **snapshot** already taken (or explicitly offline) — offline match done ([`board-sync.md`](board-sync.md))
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
| Draft | Nothing in Linear; keep `final/` on disk |

After create, write `linear_id` / `linear_url` back into `_merged/index.json` when that index exists.

---

## Scratch cleanup after publish

| Result | Action |
|--------|--------|
| Publish **fully succeeded** (all intended finals have Linear ids; epic OK unless `--no-epic`) | **Delete** `$SCRATCH_DIR` (`rm -rf` the `project-review-<RUN_ID>` dir only) |
| `--draft`, Linear down, or **any** intended final not filed | **Keep** `$SCRATCH_DIR` — required for re-file without rediscovery |

Capture epic/leaf ids and URLs for the handoff **before** deleting.  
If deleted, handoff says scratch was removed after successful file (no path needed for recovery).  
If kept, handoff **must** include the absolute scratch path.

Filtered-out discoveries (signal filter / `--p0-p1-only` never in `final/`) do not force keep. Unfiled bodies still in `final/` **do** force keep.

---

## MCP discipline

1. `search_tool` for Linear tools (create/update issue, list labels, etc.).
2. Read input schemas before calling.
3. Prefer create via `save_issue` / equivalent **without** id.
4. Use **literal newlines** in markdown descriptions (not `\\n` strings).
5. Never put secrets in titles or bodies.
6. Do **not** re-list the full board during publish for routine dedupe (snapshot already applied). On unexpected duplicate API error: mark candidate in index, continue next final.

Tool names vary by server (`linear__save_issue`, team-specific servers). Discover rather than hard-code wrong names.

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

| Review | Linear priority field |
|--------|----------------------|
| P0 data loss / broken core production path | `1` Urgent |
| Other P0 / most high-visibility P1 | `2` High |
| Remaining P1 / solid P2 | `3` Medium |
| Minor P2 | `4` Low |

(`0` = None — avoid for filed review leaves.)

---

## Labels

If `list_issue_labels` (or equivalent) succeeds:

- Apply only existing labels that clearly fit: Bug, Enhancement, UI, A11y, Content, etc.
- Do **not** invent labels.
- Omit when unsure.

---

## State & assignee

| Field | Value |
|-------|--------|
| State | Backlog, Todo, Triage, or team default for **new unstarted** work |
| Assignee | **Unassigned** |
| In Progress | **Never** |
| Done | **Never** |

`/solve` owns claim (In Progress + assignee + start comment).

---

## Create sequence (publish pass)

### A. Epic (unless `--no-epic`)

1. Create parent issue/epic with packaging body from `issue-template.md`.
2. Title: `Review pass ({mode}) – {Project or Surface} – YYYY-MM`
3. Prefer Epic type/label if the team uses it **and** the API supports it; otherwise a normal parent issue with children is fine.
4. Capture epic id + URL + identifier.
5. Prefer reusing an open “Review pass” epic from the **board snapshot** when same surface/month (see board-sync).

### B. Leaves (dependency order)

For each `final/*.md` (or ordered final list from index) in Phase 6 / D6 order:

1. Create with full issue-template body (strip YAML frontmatter if Linear should only get markdown sections — or include a clean body export field).
2. Set `team`, `project`, `priority`, `labels` (if any), `parentId`/`parent` = epic when applicable.
3. Do not assign.
4. Capture identifier, URL, id → update local index `filed`.

### C. Relations (after ids exist)

1. Set `blockedBy` for hard deps (map `blocked_by_candidates` → Linear ids via index).
2. Set `relatedTo` for `related_board` ids from offline match.
3. If API requires update calls post-create, batch them cleanly.

### D. Epic rollup note (optional)

If easy, update epic description with a bullet list of child identifiers. Not required for `/solve`.

---

## Failure handling

| Failure | Action |
|---------|--------|
| Auth error | Stop filing; **keep** full scratch package; handoff **not filed** with absolute path |
| Project mismatch | Fix project/team; retry once |
| Single leaf fails | Continue others; report failed path; **keep** scratch (partial) |
| Relations fail | Leaves still valid; note missing blockedBy in handoff; if all leaves created, may delete scratch (relations can be fixed in Linear later) — prefer **keep** if relation plan was material and incomplete |
| Unexpected duplicate on create | Mark index; do not re-crawl entire board; continue |

Never pretend issues were created when they were not.  
Never re-run full deep discovery solely because publish failed — re-file from `final/` while scratch is kept.  
Never delete scratch until publish is fully verified.

---

## Draft package format (when not filing)

Prefer pointing at disk:

```markdown
# Project review drafts — NOT FILED

Mode: deep
Team/project: …
Scratch: /tmp/grok-…/project-review-<RUN_ID>/
Finals: …/issue-candidates/final/
Index: …/issue-candidates/_merged/index.json
Epic title: …

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

## What this skill must never do on Linear

- Assign to self or others
- Set In Progress / Done / Canceled on new leaves
- Post “starting work” comments
- Mass-edit or cancel pre-existing issues
- Close the epic as Done (children incomplete)
- Create from uncleaned `_inbox` dumps
- Per-candidate board search during publish

---

## Alignment with `/solve` Linear policy

| Moment | Review | Solve |
|--------|--------|-------|
| Create backlog leaves | Yes (publish pass) | No |
| Claim / In Progress | No | Yes |
| Start/completion comments | No | Yes |
| Mark leaf Done | No | Yes after local `dev` merge |
| Epic expand / rollup Done | No | Yes when children terminal |
