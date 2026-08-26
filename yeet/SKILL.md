---
name: yeet
description: >
  Quick-ship session work with no CI babysit: fetch origin/main into local
  main, merge origin/main into local dev, commit leftover session files if
  needed, push origin/dev, open PR main ← dev, rebase-merge immediately
  with --admin (do not re-push dev after merge). Apply additive production
  migrations when the ship includes them; stop and ask on destructive schema.
  Comment Linear and mark Done when ticket ids are known. Use when the user
  runs /yeet or says "yeet it". Do not use for /prb, "push dev and PR to
  main", "ship it", babysit, or careful CI-watched merges — those stay /prb.
argument-hint: "[--via-dev-pr] [--skip-migrations] [--no-commit]"
---

# /yeet — Quick ship `dev` → `main` (no babysit)

Ship **this session’s finished work now**. `/prb` is the careful path (review gate, 15m CI watch). `/yeet` skips those.

1. Refresh `origin/main` into local `main` and `dev`
2. Commit leftover session files on `dev` if needed
3. Push `origin/dev`
4. Open or reuse PR **base `main` ← head `dev`**
5. Apply additive production migrate when the ship includes migrations
6. **Runtime proof** for in-scope ships ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)) — skip the `/prb` panel, **not** the proof
7. Merge immediately (`gh pr merge --rebase --admin`)
8. Sync local `main`. Do **not** re-push `dev` after merge.
9. Linear comment + **Done** when ticket ids are known

## Operating contract

- Integration branch is lowercase **`dev`**. Trunk is **`main`**. If this repo has no `dev` (local or `origin/dev`), **stop** — do not invent a branch.
- **Never** `git push origin dev` until `git fetch origin`, local `main` matches `origin/main` (ff-only), and `origin/main` is an ancestor of local `dev`.
- **No review gate. No CI wait. No babysit.** Merge as soon as the PR exists, **in-scope runtime proof** succeeded ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)), and migrate (if any) succeeded. `/yeet` does not run the `/prb` panel. It still must drive user-visible / auth / billing / API / schema / shared-helper ships.
- Merge with `gh pr merge --rebase --admin` so `main` lands on dev's already-pushed commits. If rebase is refused, `gh pr merge --merge --admin`. If `--admin` is denied, report the error and **stop**.
- **One dev preview + one main production per ship.** dev's preview comes from the Phase 2 dev push (or from merging `--via-dev-pr` into dev). main's production comes from the dev→main merge. Never push dev again after that merge — a dev push of the merge commit starts a second dev preview for the same ship.
- Default path is **direct on `dev`**. Do **not** open a feature-branch PR into `dev` unless `dev` is branch-protected or the user passed `--via-dev-pr`.
- **No force-push `main`.** Do not rewrite `dev` unless a non-ff push is understood and `--force-with-lease` is the only option.
- Never commit secrets, `.env*`, `.next`, `node_modules`, `.vercel`, or untracked junk such as `tmp/`.
- Never print Doppler values, tokens, or connection strings.
- Do not invent new features during `/yeet`.
- Human veto in-session cancels the merge.

## Args

| Arg | Meaning |
|-----|---------|
| `--via-dev-pr` | Commit on a short `feat/<ticket-or-slug>` branch, PR into `dev`, merge that, then PR `dev` → `main` |
| `--skip-migrations` | Do not run production migrate; loud warning that schema may lag |
| `--no-commit` | Do not auto-commit; stop if the working tree has session changes that need a commit |

Ignore unknown tokens after logging them.

## Phase 0 — Inventory

1. Confirm git root. `gh auth status` — stop if unauthenticated.
2. Record `OWNER/REPO`, current branch, `git status -sb`.
3. `git fetch origin`.
4. If neither local `dev` nor `origin/dev` exists, **stop**.
5. Session work, in order:
   1. Commits on local `dev` not on `origin/dev`
   2. Else session commits on the current branch — merge/ff them into local `dev`
   3. Else uncommitted intentional session files — Phase 0.5
6. If after 0.5 there is nothing new vs `origin/main` on `dev`, report and exit.
7. **Migrations:** if `origin/main...dev` (plus the commit you are about to make) touches this repo’s migration/schema paths, follow `/prb` [`references/db-migrations.md`](../prb/references/db-migrations.md) for discovery (`MIGRATE_CMD`, Doppler **production** config **name**, risk class). Uncommitted migration SQL must be committed in 0.5 before apply.
8. **Linear ids:** collect `SHIP_LINEAR_IDS` from `git log origin/main..dev --pretty=%B`, branch name, and the session ticket. Re-scan after commit.

Unrelated dirty paths (leave alone): `.cursor/hooks/**`, `.deepsec/`, local env files, `tmp/`.

## Phase 0.5 — Auto-commit (unless `--no-commit`)

If the working tree has no session source changes, skip.

If the dirty set looks **mixed or huge** (unrelated packages, generated junk mixed with edits, or you cannot tell what belongs to this session), **stop and ask**.

Otherwise:

1. Stage only the session source files (and their tests). Never stage the ignore list above.
2. Invent a short subject from the diff + a ticket id when known (`TEAM-123: …`). Do not ask for a subject unless the dirty set is ambiguous.
3. Commit on local `dev` (or on the `--via-dev-pr` branch).
4. Re-scan Linear ids from the new commit.

## Phase 1 — Refresh trunk into `dev`

```bash
git fetch origin
git checkout main
git merge --ff-only origin/main || git pull --ff-only origin main
# main must equal origin/main
git checkout dev 2>/dev/null \
  || git checkout -b dev origin/dev 2>/dev/null \
  || { echo "no dev branch"; exit 1; }
git merge origin/main -m "Merge origin/main into dev before /yeet"
git merge-base --is-ancestor origin/main dev   # required
```

If ff-only on `main` fails, **stop**. If the `dev` merge conflicts, resolve fully, then continue. If merging `origin/dev` is needed because remote `dev` is ahead, do that **after** `origin/main` is in, without dropping session commits.

`--via-dev-pr`: after `dev` contains `origin/main`, checkout a short `feat/<ticket-or-slug>` from `dev`, ensure the session commit is on it.

## Phase 2 — Push and PR

**Direct (default):**

```bash
git checkout dev
git push -u origin dev
```

Reuse an open PR with head `dev` and base `main`. If none:

```bash
gh pr create --base main --head dev \
  --title "<from commits>" \
  --body "$(cat <<'EOF'
## Summary
- <what this ship includes>

Opened by `/yeet`. Merging immediately; no CI wait.
EOF
)"
```

**`--via-dev-pr`:** push the feature branch, PR into `dev`, `gh pr merge --rebase --admin` (fallback `--merge --admin`), fetch, ff local `dev` to `origin/dev`, then PR `main` ← `dev` the same way. Two PRs, still no babysit. After the dev→main merge, do not re-push dev.

If `git push origin dev` is rejected non-ff, merge `origin/dev` into local `dev` (keep session commits), re-merge `origin/main` if main moved, push again. `--force-with-lease` only when this is exclusively the user’s integration branch and the divergence is understood.

Title/body must match `git log origin/main..dev --oneline`.

## Phase 2.5 — Runtime proof (in-scope)

**Authority:** [`../docs/prove-it-works.md`](../docs/prove-it-works.md).

After the PR exists (or before merge if the PR is reused), if `origin/main...dev` is in-scope (UI, auth, billing, public API, schema, shared helper):

1. Drive the real path. Visual parity if a ticket/session named a reference. Blast-radius run if shared/schema/auth.
2. Fail → **do not merge**. Report what was unproven. Do not treat typecheck as a pass.
3. Out of scope (docs-only, comments-only): skip; note `Runtime: n/a` in the report.

## Phase 3 — Migrate (when in ship)

Skip when no migration paths or `--skip-migrations` (warn).

Authority: `/prb` [`references/db-migrations.md`](../prb/references/db-migrations.md).

- **Additive / expand:** run this project’s production migrate command **before** merge.
- **Destructive or unknown:** **do not merge**. Stop and ask.
- Never `db:push` / schema-push to production. Never print secrets. Never seed.

Preview migrate is optional and only when the project documents a preview/stg config that would break without schema.

## Phase 4 — Merge now

```bash
gh pr merge $PR_NUMBER --rebase --admin \
  || gh pr merge $PR_NUMBER --merge --admin
git fetch origin
git checkout main
git merge --ff-only origin/main
git checkout dev
git merge --ff-only origin/main || true
# Do not `git push origin dev` here.
```

Rebase-merge so `main` fast-forwards onto dev's already-pushed commits when it can. dev already got its preview from Phase 2. Pushing dev after merge (to carry GitHub's merge commit) starts a second dev preview — skip it. If dev cannot fast-forward (rebase rewrote commits, or `--merge` left dev one merge-commit behind `main`), leave dev. Next `/yeet` Phase 1 merges `origin/main` into dev before the next dev push.

Do not `reset --hard` if it would destroy unique local commits. If `--admin` fails, report the exact `gh` error and stop.

## Phase 5 — Linear (when ids exist)

Keep the comment short. Read-before-write: [`../docs/linear-comments.md`](../docs/linear-comments.md).

For each id in `SHIP_LINEAR_IDS` that this ship implemented:

1. **`list_comments` first.** Skip if `/prb — shipped to production` or `/yeet` shipped already exists for this PR or merge SHA.
2. Else comment: `/yeet` shipped — PR number + URL, merge SHA, migrate note (config **name** only).
3. Mark **Done**.
4. Skip ids still In Progress with a **foreign** live `claimed-by:` comment.

If Linear is down, dump the comment body in the user report. Do not fail the ship.

## Report

```markdown
** /yeet complete**
**Repo:** owner/repo
**Commit:** created `<subject>` | none | skipped (--no-commit) | blocked (dirty set)
**Pushed:** origin/dev @ <sha>
**PR:** #N — <url> (main ← dev)
**Runtime proof:** driven `<path>` → `<observed>` | n/a | blocked
**Merge:** merged @ <sha> | blocked (<reason>)
**Migrations:** none | applied production (`<config>`) | blocked | skipped
**Linear:** Done on TEAM-123 | none | skipped foreign In Progress
**Local:** main synced to origin/main; dev not re-pushed after merge
```

## Anti-patterns

- Waiting for CI or starting a `/prb` babysit
- Merging an in-scope ship because `/yeet` skips the review panel (proof is still required)
- Opening a feature-branch PR into `dev` when not asked and `dev` is not protected
- Pushing `dev` that does not contain `origin/main`
- Re-pushing `dev` after dev→main merge just to carry a merge commit (second dev preview)
- Force-pushing `main`
- Committing `tmp/`, `.env*`, or unrelated dirt
- Merging a ship that needs new tables without this repo’s production migrate
- Stealing `/prb` when the user said “ship it” or “push dev and PR to main”
- Inventing a `dev` branch in a repo that does not have one
- Posting a Linear ship comment without `list_comments` first
