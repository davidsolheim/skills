# Linear ship comments (`/prb`)

After work ships through `/prb`, **every Linear issue that was part of the ship**
gets a short comment with PR + deployment evidence so anyone can see what went
live and how to find it.

Comment shape lives here. **Done after merge** is `SKILL.md` Phase 4.5.
Read-before-write: [`../../docs/linear-comments.md`](../../docs/linear-comments.md).

---

## When to comment

| Moment | Required? | What to say |
|--------|-----------|-------------|
| **Phase 1.5** — local review gate exits | Yes (when a ship id exists) | Template C on the **first** ship issue only |
| **Phase 2** — PR opened or reused | Yes (when issues known) | PR number + URL; head `dev` → base `main`; “in ship / awaiting merge” |
| **Phase 4.5** — after **merge to `main`** | Yes | Production ship note: PR, merge SHA, **deployment id + URL** when available |
| Mid babysit / CI only | No | Do not spam per tick |

If merge is blocked or `--no-merge`, still leave the **PR comment** when the PR
exists. Skip the production deployment comment until merge (+ deploy) actually
happens (or run it later via `/prb check` / resume after human merge).

---

## Which issues are “part of the deployment”

Build `SHIP_LINEAR_IDS` (unique, sorted) from:

1. **Commit messages** on the ship set:
   ```bash
   git log origin/main..dev --pretty=%B
   # also after merge: git log <base>..<merge_sha> --pretty=%B
   ```
   Match issue keys: `\b([A-Z][A-Z0-9]+-\d+)\b` (e.g. `TW-123`, `INV-45`, `BNF-9`).
2. **Branch names** for session issue branches merged into this ship (if still
   visible in merge commits).
3. **PR body / title** if they already mention issue ids.
4. Optional: `gh pr view $PR_NUMBER --json commits,body,title` after PR exists.

Phase 1.5 does **not** mint Linear issues. Gate findings live in scratch. The
gate still posts **one** template C comment on the first ship id.

**Include** parent epic ids only if the epic id itself appears in commits or the
user listed it — do **not** auto-comment every sibling under an epic.

**Deduplicate** before posting. Cap noise: if more than **40** ids, comment on
all of them but report count; do not invent a “batch only” epic substitute
unless Linear tooling fails.

**Skip** false positives carefully: bare words that match poorly — prefer
`PREFIX-digits` with a team-like prefix (2+ letters).

---

## Linear MCP

1. `search_tool` for Linear comment tools on the correct server
   (`linear`, or the workspace-specific Linear MCP server) matching the repo’s team.
2. **Before every `save_comment`:** `list_comments` on that issue (recent first).
   Skip posting when a comment already covers this **moment + PR**:
   - Template A: heading `/prb — in ship` and this `PR_NUMBER`
   - Template B: heading `/prb — shipped to production` and this `PR_NUMBER`
     (or this `MERGE_SHA`)
   - Template C: heading `/prb — local review gate` and this `dev` SHA
   A leftover babysit tick, a second `/prb` on the same PR, and the
   orchestrator racing a scheduler are the same skip. Do not “improve” an
   existing matching comment by posting another.
3. Create with `*_save_comment` only after step 2 says the moment is missing:
   - `issueId`: `TEAM-123` (identifier is fine)
   - `body`: markdown with **literal newlines** (not `\n` escapes)
4. Do **not** set assignee/state as part of this comment flow (Done is Phase 4.5
   in `SKILL.md` after merge, and still skip if template B already exists).
5. If one issue fails (not found / wrong workspace): log it, continue others.
6. If Linear is fully down: put the same bodies in the Phase 5 user report under
   **Linear comments (not posted)** so nothing is lost.

One comment per issue per moment (gate / PR open / production). Template C
is once per run on the first ship id. Count a skip as posted for the report
(`LINEAR_*_COMMENTS_POSTED` includes skipped-as-already-present).

---

## Comment templates

### A. PR opened / in ship (Phase 2)

```markdown
## /prb — in ship

- **PR:** [#N](PR_URL) (`dev` → `main`)
- **Repo:** `owner/repo`
- **Head SHA:** `abc1234`
- **Status:** opened/reused by `/prb`; babysit + merge pending (unless already merged)

This issue’s commits are included in the ship set for this PR.
```

### B. Production live (Phase 4.5 — after merge)

```markdown
## /prb — shipped to production

- **PR:** [#N](PR_URL) — **merged**
- **Merge commit:** `full_or_short_sha` (on `main`)
- **Repo:** `owner/repo`
- **Production deployment ID:** `dpl_…` | `N/A (not found)`
- **Production deployment URL:** https://… | project dashboard link | `N/A`
- **Inspector / dashboard:** <Vercel deployment URL or host equivalent>
- **Deploy target:** production
- **Migrated (if any):** yes (`<config name only>`) | none in ship | skipped
- **Shipped at:** <ISO UTC>

Pushed live via `/prb` (merge to `main` + production deploy when applicable).
```

Keep it short. No secrets, no Doppler values, no connection strings.

If deployment id is still building, comment once with merge facts and:

```markdown
- **Production deployment ID:** pending at comment time — re-check Vercel/host for commit `sha`
```

Prefer waiting briefly (see deploy discovery) so the first production comment
includes a real id when the platform is fast.

### C. Local review gate (Phase 1.5 — once per `/prb` run)

Post on the **first** `SHIP_LINEAR_ID` after the gate exits (`clean`,
`blocked-at-cap`, or `skipped` with proof). Skip if no ship ids — dump the
body in the Phase 5 report instead.

```markdown
## /prb — local review gate

- **Repo:** `owner/repo`
- **dev SHA:** `abc1234`
- **Ship intensity:** <band>
- **Cycles:** N
- **Actionable found:** F1 … (or none)
- **Fixed:** <one line per finding, or none>
- **Leftover:** none | F… (blocked at cap — **did not push**)
- **Runtime proof:** driven `<path>` → `<observed>` | n/a

Scratch review files were not committed.
```

One comment per `dev` SHA. Do not post C on every ship id. Do not file Linear
issues for F1…Fn.

---

## Deployment discovery (production)

After `gh pr merge` succeeds and local `main` is updated to the merge SHA:

### Prefer in order

1. **GitHub deployment / check** for the merge commit on `main`:
   ```bash
   gh api repos/OWNER/REPO/deployments?sha=MERGE_SHA&environment=production --jq '.[0]'
   # or
   gh api repos/OWNER/REPO/commits/MERGE_SHA/status
   gh pr checks <merged PR>   # may still show deploy checks
   ```
2. **Vercel CLI** (when project is linked / user is logged in):
   ```bash
   vercel ls --prod 2>/dev/null | head
   vercel inspect <url-or-id> 2>/dev/null
   # Or list deployments for the project and match meta githubCommitSha == MERGE_SHA
   ```
3. **Vercel MCP** / dashboard API if available in-session — same goal: id + url
   for the **production** deployment of `MERGE_SHA`.
4. **Host-specific docs** in `AGENTS.md` (Railway, Fly, etc.) — follow project
   commands; still capture an id/url/name for the comment.

### Fields to capture

| Field | Example |
|-------|---------|
| `DEPLOY_ID` | `dpl_6A7B…` or platform uuid |
| `DEPLOY_URL` | `https://project-….vercel.app` or production domain |
| `DEPLOY_INSPECT` | `https://vercel.com/…/dpl_…` |
| `DEPLOY_ENV` | `production` |
| `MERGE_SHA` | full git sha on `main` |
| `PR_NUMBER` / `PR_URL` | from Phase 2 |

If no host deploy applies (library-only repo, no CD): set deploy fields to
`n/a (no production deploy for this repo)` and still comment with PR + merge SHA.

### Timing

- Wait up to **~2–3 minutes** polling for a production deployment matching
  `MERGE_SHA` (short sleeps). Do not block forever.
- If still pending: post comment with merge + `deployment: pending` rather than
  skipping the comment entirely.

---

## Tracking fields (for Phase 5 report)

```text
SHIP_LINEAR_IDS=TEAM-1,TEAM-2,…
LINEAR_PR_COMMENTS_POSTED=N
LINEAR_SHIP_COMMENTS_POSTED=N
LINEAR_COMMENT_FAILURES=TEAM-x (reason), …
DEPLOY_ID=…
DEPLOY_URL=…
MERGE_SHA=…
```

Report line example:

```markdown
**Linear ship comments:** PR notes on K issues · production notes on K issues · deploy `dpl_…` · failures: none
```
