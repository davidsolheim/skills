---
name: yeet
description: >
  Quick-ship session work with no review babysit: fetch origin/main into local
  main, merge origin/main into local dev, commit leftover session files if
  needed, push origin/dev, open PR main ← dev, rebase-merge immediately
  with --admin (do not re-push dev after merge). Watch GitHub/Vercel **build**
  of the ship SHA until Ready or Error (do not merge / do not claim complete
  on Error). Apply additive production migrations when the ship includes them;
  stop and ask on destructive schema. The WCP issue file is already the close
  record. After origin/dev, record the dev SHA in that repo's Notion issues
  database. When the production build of origin/main is Ready, set Status done.
  Do not call Linear. Use when the user runs /yeet or says "yeet it". Do not use
  for /prb, "push dev and PR to main", "ship it", babysit, or careful
  CI-watched merges — those stay /prb.
argument-hint: "[--via-dev-pr] [--skip-migrations] [--no-commit]"
---

# /yeet — Quick ship `dev` → `main` (no babysit)

Ship **this session’s finished work now**. `/prb` is the careful path (deep local review panel + 15m CI watch). `/yeet` skips those. Cheap `/solve` has no inner review swarm, so `/yeet` after `/solve` is an explicit skip of the only remaining audit.

1. Refresh `origin/main` into local `main` and `dev`
2. Commit leftover session files on `dev` if needed
3. Push `origin/dev`
4. Open or reuse PR **base `main` ← head `dev`**
5. Apply additive production migrate when the ship includes migrations
6. **Runtime proof** for in-scope ships ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)) — skip the `/prb` panel, **not** the proof
7. **Watch the GitHub/Vercel build of the `dev` SHA.** Error → do not merge.
8. Merge immediately (`gh pr merge --rebase --admin`)
9. Sync local `main`. Do **not** re-push `dev` after merge.
10. **Watch the production build of the merge SHA.** Error → do not set Notion `done`; fix or report.
11. When that production build is Ready, set Notion Status `done` immediately ([`../docs/notion-issues.md`](../docs/notion-issues.md)). The dev push already recorded Dev SHA. Do not call Linear.

## Operating contract

- Integration branch is lowercase **`dev`**. Trunk is **`main`**. If this repo has no `dev` (local or `origin/dev`), **stop** — do not invent a branch.
- **Never** `git push origin dev` until `git fetch origin`, local `main` matches `origin/main` (ff-only), and `origin/main` is an ancestor of local `dev`.
- **WCP:** this skill is the human export ([`../docs/wcp.md`](../docs/wcp.md)). `wcp look` before push; wait if live leases remain. **`unset WCP_AGENT` immediately before `git push origin dev`**. Commit `.WCP/issues/`. Do not commit `.WCP/RUN.md`, `.WCP/run.sqlite`, or sqlite wal/shm.
- **No review gate. No `/prb` babysit** of bot comments. **Do wait for the ship build.** After the `dev` push, poll GitHub Actions `build` (or this repo’s compile job) and the Vercel preview for that SHA until success or failure (cap ~10 minutes). After merge, poll the Vercel **production** deploy (or GitHub `build` on `main`) the same way. Error → **do not merge** (preview/CI) or **do not claim complete / do not set Notion done** (production). `/yeet` does not run the `/prb` panel. It still must drive user-visible / auth / billing / API / schema / shared-helper ships ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)).
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
| `--via-dev-pr` | Commit on a short `david/…` branch, PR into `dev`, merge that, then PR `dev` → `main` |
| `--skip-migrations` | Do not run production migrate; loud warning that schema may lag |
| `--no-commit` | Do not auto-commit; stop if the working tree has session changes that need a commit |
| `--skip-build-watch` | Do not wait for GitHub/Vercel build; loud warning. Default is to watch. |

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
8. **Ship issues:** collect `SHIP_ISSUE_IDS` per [`../docs/notion-issues.md`](../docs/notion-issues.md) from `.WCP/issues/` and `git log origin/main..dev --pretty=%H%n%s`. Re-scan after commit.

Unrelated dirty paths (leave alone): `.cursor/hooks/**`, `.deepsec/`, local env files, `tmp/`.

## Phase 0.5 — Auto-commit (unless `--no-commit`)

`wcp look --json` first ([`../docs/wcp.md`](../docs/wcp.md)). If a live source-file lease remains, wait or stop. Do not commit another writer's burst and do not stash it. Stage `.WCP/issues/` with the work. Do not stage `.WCP/RUN.md`, `.WCP/run.sqlite`, or sqlite wal/shm.

If the working tree has no session source changes, skip.

If the dirty set looks **mixed or huge** (unrelated packages, generated junk mixed with edits, or you cannot tell what belongs to this session), **stop and ask**.

Otherwise:

1. Stage only the session source files (and their tests). Never stage the ignore list above.
2. Invent a short subject from the diff + the issue id when known (`0123: …`). Do not ask for a subject unless the dirty set is ambiguous.
3. Commit on local `dev` (or on the `--via-dev-pr` branch).
4. Re-scan `SHIP_ISSUE_IDS` from the new commit.

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

`--via-dev-pr`: after `dev` contains `origin/main`, checkout a short `david/<ticket-or-slug>` from `dev`, ensure the session commit is on it.

## Phase 2 — Push and PR

**Direct (default):**

```bash
git checkout dev
unset WCP_AGENT WCP_NAME_TOKEN
git push -u origin dev
```

Reuse an open PR with head `dev` and base `main`. If none:

```bash
gh pr create --base main --head dev \
  --title "<from commits>" \
  --body "$(cat <<'EOF'
## Summary
- <what this ship includes>

Opened by `/yeet`. Merging after the ship build is green; no review babysit.
EOF
)"
```

**`--via-dev-pr`:** push the feature branch, PR into `dev`, `gh pr merge --rebase --admin` (fallback `--merge --admin`), fetch, ff local `dev` to `origin/dev`, then PR `main` ← `dev` the same way. Two PRs, still no babysit. After the dev→main merge, do not re-push dev.

If `git push origin dev` is rejected non-ff, merge `origin/dev` into local `dev` (keep session commits), re-merge `origin/main` if main moved, push again. `--force-with-lease` only when this is exclusively the user’s integration branch and the divergence is understood.

Title/body must match `git log origin/main..dev --oneline`.

Immediately after this push, run the Notion **on dev** update ([`../docs/notion-issues.md`](../docs/notion-issues.md)): Dev SHA and PR URL. Do not set `done`.

## Phase 2.5 — Runtime proof (in-scope)

**Authority:** [`../docs/prove-it-works.md`](../docs/prove-it-works.md).

After the PR exists (or before merge if the PR is reused), if `origin/main...dev` is in-scope (UI, auth, billing, public API, schema, shared helper):

1. Drive the real path. Visual parity if a ticket/session named a reference. Blast-radius run if shared/schema/auth.
2. Fail → **do not merge**. Report what was unproven. Do not treat typecheck as a pass.
3. Out of scope (docs-only, comments-only): skip; note `Runtime: n/a` in the report.

## Phase 2.6 — Watch the `dev` SHA build (default)

Do **not** merge until this passes. Skip only for docs-only / comments-only ships.

Poll until **success or failure** (not until a 15-minute review window). Cap **10 minutes**. Do not watch Copilot/CodeRabbit/review comments. Do not wait for the GitHub **Test**/**Lint** steps — those are `/prb`. This phase is **compile/deploy**.

1. `HEAD_SHA` = `origin/dev` after the Phase 2 push.
2. **Vercel (required when the repo deploys there):** `vercel ls` / `vercel inspect <preview-url> --wait` for the Preview of `$HEAD_SHA`. **Error** is a fail. **Ready** is the pass.
3. If GitHub Actions has a compile step (`bun run build` / `next build`) that failed on `$HEAD_SHA`, treat that as the same fail even if you have not seen Vercel yet — pull the TypeScript/log error.
4. On **failure**: do **not** merge. Pull the failed build log, fix, commit on `dev`, push, re-watch. Do not set Notion `done`.
5. On **timeout** with no conclusion: do **not** merge. Report the last status.

`--skip-build-watch` (only if the user passed it): skip this phase; loud warning.

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

## Phase 4.5 — Watch the production build (default)

Same rules as Phase 2.6, for **`$MERGE_SHA` on `main`**.

1. Poll `vercel inspect` of the **Production** deployment for `$MERGE_SHA` until Ready or Error (cap ~10 minutes). GitHub compile-step failure on `main` is the same fail.
2. **Error** → do **not** set Notion `done`. Fetch the log, fix on `dev`, ship again. Report **Build:** failed.
3. **Ready** → immediately set Notion Status `done` and Main SHA ([`../docs/notion-issues.md`](../docs/notion-issues.md)). Then Phase 5.
4. Timeout with no conclusion → do not set `done`; report last status.

## Phase 5 — Queue

If a shipped issue file is still `open` or `in-progress` and its work commit is in this ship, set `commit` and `status: done` and move it to `done/` ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Skip a file another agent holds under a live lease. Notion Status is already `done` from Phase 4.5 when the production build is Ready. If this step moves a file that was missing from that update, upsert it to `done` now. Do not call Linear.

## Report

```markdown
** /yeet complete**
**Repo:** owner/repo
**Commit:** created `<subject>` | none | skipped (--no-commit) | blocked (dirty set)
**Pushed:** origin/dev @ <sha>
**PR:** #N — <url> (main ← dev)
**Runtime proof:** driven `<path>` → `<observed>` | n/a | blocked
**Merge:** merged @ <sha> | blocked (<reason>)
**Build:** preview Ready @ <sha> · production Ready @ <sha> | failed (`<url>`) | timeout | skipped
**Migrations:** none | applied production (`<config>`) | blocked | skipped
**Notion:** done on 0123 | dev SHA only | none | skipped (live lease) | skipped (build failed)
**Local:** main synced to origin/main; dev not re-pushed after merge
```

## Anti-patterns

- Starting a `/prb` babysit of review-bot comments
- Merging while Vercel preview/production (or `next build`) for the ship SHA is **Error** or still running
- Setting Notion **done** before the production build is Ready
- Merging an in-scope ship because `/yeet` skips the review panel (proof is still required)
- Opening a feature-branch PR into `dev` when not asked and `dev` is not protected
- Pushing `dev` that does not contain `origin/main`
- Re-pushing `dev` after dev→main merge just to carry a merge commit (second dev preview)
- Force-pushing `main`
- Committing `tmp/`, `.env*`, or unrelated dirt
- Merging a ship that needs new tables without this repo’s production migrate
- Stealing `/prb` when the user said “ship it” or “push dev and PR to main”
- Inventing a `dev` branch in a repo that does not have one
- Calling Linear for ship status
