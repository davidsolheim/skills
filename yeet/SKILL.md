---
name: yeet
description: >
  Quick-ship session work: fetch origin/main into local main, merge origin/main
  into local dev, commit leftover session files if needed, push origin/dev,
  open PR main ← dev, rebase-merge immediately with --admin (do not re-push
  dev after merge just to carry the merge commit). Watch the GitHub/Vercel
  build of the ship SHA until Ready or Error (do not merge / do not claim
  complete on Error). After that production build is Ready, watch the PR for
  Codex and other review-agent comments and fix P0–P2 with a follow-up ship.
  Apply additive production migrations when the ship includes them; stop and
  ask on destructive schema. The wcp issue file is already the close record.
  After origin/dev, record the dev SHA in that repo's Notion issues database.
  When the production build of origin/main is Ready, set Status done. Do not
  call Linear. Use when the user runs /yeet or says "yeet it". Do not use for
  /prb, "push dev and PR to main", "ship it", babysit, or careful CI-watched
  merges — those stay /prb.
argument-hint: "[--via-dev-pr] [--skip-migrations] [--no-commit] [--skip-review-watch]"
---

# /yeet — Quick ship `dev` → `main`, then fix review-agent P0–P2

Ship **this session’s finished work now**. `/prb` is the careful path (local review panel + 15m pre-merge CI watch). `/yeet` skips that panel and does not wait to merge. After `main` is updated and the production build is Ready, watch the merged PR and fix P0–P2 findings from Codex or any other review agent.

1. Refresh `origin/main` into local `main` and `dev`
2. Commit leftover session files on `dev` if needed
3. Push `origin/dev`
4. Open or reuse PR **base `main` ← head `dev`**
5. Apply additive production migrate when the ship includes migrations
6. **Runtime proof** for in-scope ships ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)) — skip the `/prb` panel, **not** the proof
7. **Watch the GitHub/Vercel build of the `dev` SHA.** Error → do not merge.
8. Merge immediately (`gh pr merge --rebase --admin`)
9. Sync local `main`. Do **not** re-push `dev` just to carry the merge commit.
10. **Watch the production build of the merge SHA.** Error → do not set Notion `done`; fix or report.
11. When that production build is Ready, mirror the main sha to the tracker named in `.wcp/tracker.md`. The dev push already recorded the dev sha. Skip when `tracker` is `none` or the file is missing. Notion uses [`../docs/notion-issues.md`](../docs/notion-issues.md).
12. **Watch that PR for review-agent P0–P2** (Phase 4.6). Fix them with a follow-up ship. Do not start this watch before the merge.

## Operating contract

- Integration branch is lowercase **`dev`**. Trunk is **`main`**. If this repo has no `dev` (local or `origin/dev`), **stop** — do not invent a branch.
- **Never** `git push origin dev` until `git fetch origin`, local `main` matches `origin/main` (ff-only), and `origin/main` is an ancestor of local `dev`.
- **wcp:** follow [`../docs/wcp.md`](../docs/wcp.md). If any issue is in `.wcp/issues/in-progress/`, wait. Do not commit or stash that work. Commit `.wcp/issues/`. After the push to `origin/dev`, move the shipped `done/` issues to `deployed-dev/` and write the sha in `dev`. After the commit is on `origin/main`, move them to `deployed-main/` and write the sha in `main`.
- **No pre-merge review gate. No `/prb` babysit.** Do not delay the merge for review comments, Test, or Lint. **Do wait for the ship build.** After the `dev` push, poll GitHub Actions `build` (or this repo’s compile job) and the Vercel preview for that SHA until success or failure (cap ~10 minutes). After merge, poll the Vercel **production** deploy (or GitHub `build` on `main`) the same way. Error → **do not merge** (preview/CI) or **do not claim complete / do not set Notion done** (production). `/yeet` does not run the `/prb` panel. It still must drive user-visible / auth / billing / API / schema / shared-helper ships ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)). After production is Ready, Phase 4.6 watches review-agent comments and fixes P0–P2.
- Merge with `gh pr merge --rebase --admin` so `main` lands on dev's already-pushed commits. If rebase is refused, `gh pr merge --merge --admin`. If `--admin` is denied, report the error and **stop**.
- **One dev preview + one main production per ship.** dev's preview comes from the Phase 2 dev push (or from merging `--via-dev-pr` into dev). main's production comes from the dev→main merge. Never push dev again after that merge just to carry the merge commit — that starts a second dev preview of the same ship. A Phase 4.6 fix is a new ship and does push `dev`.
- Default path is **direct on `dev`**. Do **not** open a feature-branch PR into `dev` unless `dev` is branch-protected or the user passed `--via-dev-pr`.
- **No force-push `main`.** Do not rewrite `dev` unless a non-ff push is understood and `--force-with-lease` is the only option.
- Never commit secrets, `.env*`, `.next`, `node_modules`, `.vercel`, or untracked junk such as `tmp/`.
- Never print Doppler values, tokens, or connection strings.
- Do not invent new features during `/yeet`.
- Human veto in-session cancels the merge and any Phase 4.6 follow-up ship.

## Args

| Arg | Meaning |
|-----|---------|
| `--via-dev-pr` | Commit on a short `david/…` branch, PR into `dev`, merge that, then PR `dev` → `main` |
| `--skip-migrations` | Do not run production migrate; loud warning that schema may lag |
| `--no-commit` | Do not auto-commit; stop if the working tree has session changes that need a commit |
| `--skip-build-watch` | Do not wait for GitHub/Vercel build; loud warning. Default is to watch. |
| `--skip-review-watch` | Do not run Phase 4.6; loud warning. Default is to watch and fix P0–P2. |

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
8. **Ship issues:** collect `SHIP_ISSUE_IDS` per [`../docs/notion-issues.md`](../docs/notion-issues.md) from `.wcp/issues/` and `git log origin/main..dev --pretty=%H%n%s`. Re-scan after commit.

Unrelated dirty paths (leave alone): `.cursor/hooks/**`, `.deepsec/`, local env files, `tmp/`.

## Phase 0.5 — Auto-commit (unless `--no-commit`)

Read `.wcp/issues/in-progress/` first ([`../docs/wcp.md`](../docs/wcp.md)). If any issue is there, wait or stop. Do not commit another writer's work and do not stash it. Stage `.wcp/issues/` with the work. Do not stage any other path under `.wcp/`.

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
git push -u origin dev
```

Reuse an open PR with head `dev` and base `main`. If none:

```bash
gh pr create --base main --head dev \
  --title "<from commits>" \
  --body "$(cat <<'EOF'
## Summary
- <what this ship includes>

Opened by `/yeet`. Merging after the ship build is green. Post-merge: fix review-agent P0–P2.
EOF
)"
```

**`--via-dev-pr`:** push the feature branch, PR into `dev`, `gh pr merge --rebase --admin` (fallback `--merge --admin`), fetch, ff local `dev` to `origin/dev`, then PR `main` ← `dev` the same way. Two PRs, still no pre-merge babysit. After the dev→main merge, Phase 4.6 watches the main PR. Do not re-push dev except for that fix.

If `git push origin dev` is rejected non-ff, merge `origin/dev` into local `dev` (keep session commits), re-merge `origin/main` if main moved, push again. `--force-with-lease` only when this is exclusively the user’s integration branch and the divergence is understood.

Title/body must match `git log origin/main..dev --oneline`.

Immediately after this push, mirror the dev sha when `.wcp/tracker.md` names a tracker. Do not mark it shipped to main. Skip when `tracker` is `none` or the file is missing.

## Phase 2.5 — Runtime proof (in-scope)

**Authority:** [`../docs/prove-it-works.md`](../docs/prove-it-works.md).

After the PR exists (or before merge if the PR is reused), if `origin/main...dev` is in-scope (UI, auth, billing, public API, schema, shared helper):

1. Drive the real path. Visual parity if a ticket/session named a reference. Blast-radius run if shared/schema/auth.
2. Fail → **do not merge**. Report what was unproven. Do not treat typecheck as a pass.
3. Out of scope (docs-only, comments-only): skip; note `Runtime: n/a` in the report.

## Phase 2.6 — Watch the `dev` SHA build (default)

Do **not** merge until this passes. Skip only for docs-only / comments-only ships.

Poll until **success or failure** (not until a review window). Cap **10 minutes**. Do not read review comments in this phase — Phase 4.6 does that after production is Ready. Do not wait for the GitHub **Test**/**Lint** steps — those are `/prb`. This phase is **compile/deploy**.

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

Rebase-merge so `main` fast-forwards onto dev's already-pushed commits when it can. dev already got its preview from Phase 2. Pushing dev after merge (to carry GitHub's merge commit) starts a second dev preview — skip that push. Phase 4.6 may push later, and only with new fix commits. If dev cannot fast-forward (rebase rewrote commits, or `--merge` left dev one merge-commit behind `main`), leave dev. Next `/yeet` Phase 1 merges `origin/main` into dev before the next dev push.

Do not `reset --hard` if it would destroy unique local commits. If `--admin` fails, report the exact `gh` error and stop.

## Phase 4.5 — Watch the production build (default)

Same rules as Phase 2.6, for **`$MERGE_SHA` on `main`**.

1. Poll `vercel inspect` of the **Production** deployment for `$MERGE_SHA` until Ready or Error (cap ~10 minutes). GitHub compile-step failure on `main` is the same fail.
2. **Error** → do **not** set Notion `done`. Fetch the log, fix on `dev`, ship again. Report **Build:** failed.
3. **Ready** → immediately set Notion Status `done` and Main SHA ([`../docs/notion-issues.md`](../docs/notion-issues.md)). Then Phase 4.6. Do not wait for review comments before this `done`.
4. Timeout with no conclusion → do not set `done`; report last status. Do not start Phase 4.6.

## Phase 4.6 — Review-agent P0–P2 after a safe merge

**When:** Phase 4 merged, and Phase 4.5 production is **Ready**. Skip if the merge did not happen, production is not Ready, or the user passed `--skip-review-watch` (loud warning).

Do not delay Phase 4 for this. Do not run the `/prb` panel. Do not treat Test, Lint, nits, or human comments as something to fix here.

### What counts

Read all three surfaces on the merged PR. Paginate. A merged PR still receives comments. Include comments already posted during the build watch.

1. Inline: `gh api --paginate repos/$OWNER/$REPO/pulls/$PR/comments`
2. Review bodies: `gh api --paginate repos/$OWNER/$REPO/pulls/$PR/reviews`
3. Conversation: `gh api --paginate repos/$OWNER/$REPO/issues/$PR/comments`

Strip HTML tags and `<!-- ... -->` before matching. A shields badge counts: alt text `P2 Badge` and URL `badge/P2-` are the same tag. Codex posts one finding per inline comment. Its review-body blurb (“Codex Review”, no priority tag) is not a finding.

A post is in scope when **both** are true:

- The author is a review agent: `chatgpt-codex-connector`, any `user.type == Bot` or login ending in `[bot]` that posted a code review, or another review app (Copilot, CodeRabbit, Cursor Bugbot, Greptile, and the same kind of reviewer). Match the post, not a vendor allowlist.
- The stripped text has a priority tag **P0, P1, or P2**. A tag is `P0 Badge` / `P1 Badge` / `P2 Badge`, a shields path `badge/P0` `badge/P1` `badge/P2`, a bracket `[P0]` `[P1]` `[P2]`, or a heading `P0`/`P1`/`P2` followed by `:` or a dash. A bare `P2` inside a sentence or a log does not count. Same bar as [`../prb/references/review-rubric.md`](../prb/references/review-rubric.md): those three are actionable. **P3** and below are not.

Leave, and list in the report: P3 and below, untagged suggestions, deploy/CI chatter (Vercel ready, Actions status) with no P0–P2 tag, and human comments. Do not fix those in this phase.

Skip a thread whose newest reply already cites a fix SHA that is on `origin/main`.

### Watch

Read once as soon as production is Ready. Then poll every 30–60 seconds until **15 minutes** after that Ready. If a review-agent check was still running at that mark, wait until it finishes or **10 more minutes**, whichever comes first.

Fix on the first in-scope batch. Do not sit out the rest of the window before fixing. After that follow-up ship’s production build is Ready, watch the new PR the same way. Cap **two** follow-up ships. Findings after that cap are reported and left.

Prefer a `monitor` that prints only `ACTION_REQUIRED` (new in-scope finding) or `DONE` (window over, nothing in scope). Do not print each poll. Do not emit the yeet report before this phase ends.

### Fix

A fix is a **new ship** of new commits. It is not a push of the merge commit.

1. `git fetch origin`. Fast-forward local `main` to `origin/main`. Check out `dev`. Merge `origin/main` into `dev`.
2. Fix the defect against the current tree. Do not apply a stale hunk blindly. Edit only what those findings require. Follow [`../docs/wcp.md`](../docs/wcp.md). Read `in-progress/` before the commit. Do not commit another writer’s work.
3. One commit for the batch. Subject names the PR and the tags (`fix: PR #N P2 …`).
4. `git push origin dev`. This push is required.
5. Open a **new** PR `main` ← `dev` (the previous PR is merged). Run Phases 2.5 through 4.5 on it. Phase 3 runs only when the fix touches schema.
6. After that fix SHA is on `origin/main`, reply on each fixed thread with that SHA, then resolve the thread. Do not reply before the SHA is on `main`. No “will fix” reply.
7. False positive, already fixed, or not introduced by this ship: reply with the reason, do not edit, resolve when the reply succeeds.

If the fix ship’s build fails, do not merge it. Report **Review:** blocked. Leave the original Notion `done` in place.

When the follow-up production build is Ready, set **Main SHA** on the same ship ids to that new SHA. Do not move Notion Status backward.

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
**Review:** none | fixed <P0×n P1×n P2×n> in #<follow-up> @ <sha> | left <P3 or untagged count> | skipped (--skip-review-watch) | blocked (<reason>)
**Migrations:** none | applied production (`<config>`) | blocked | skipped
**Notion:** done on 0123 | dev SHA only | none | skipped (live lease) | skipped (build failed)
**Local:** main synced to origin/main; no merge-commit re-push | fix pushed @ <sha>
```

## Anti-patterns

- Delaying the merge to babysit review comments, Test, or Lint (that is `/prb`)
- Skipping Phase 4.6, or fixing only Codex when another review agent posted a P0–P2
- Treating P3 or an untagged suggestion as a fix
- Refusing to push `dev` for a Phase 4.6 fix
- Replying “will fix” before the fix SHA is on `origin/main`
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
