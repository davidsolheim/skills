---
name: prb
description: >
  Ship session work: fetch origin/main into local main, merge origin/main into
  local dev, run an intensity-selected Local Code Review Gate on grok-4.6
  (scratch markdown + one fixer on local dev until clean; one Linear gate
  comment), then push to origin/dev; open a PR from dev into main; babysit
  CI/bot feedback on a 5-minute cadence for up to 15 minutes; auto-merge into
  main only if no useful automated comments or CI failures appear. When the
  ship includes DB migrations, discover and run this project's production
  migration procedure at the correct pre-merge gate (never invent a stack;
  never db:push to prod by default). After PR open and again after production
  ship, comment on every Linear issue in the ship set with PR URL, merge SHA,
  and production deployment id/URL when available. Use when the user runs
  /prb, says "push dev and PR to main", "ship session work", "babysit then
  merge to main", "promote dev to main with CI watch", or production
  migrate-on-ship.
argument-hint: "[--no-merge] [--skip-migrations] [--skip-ship-build] [--skip-review] [--exhaustive-review|--no-exhaustive] [--max-fix-cycles N] [--watch-minutes N] [--interval-minutes M]"
---

# /prb — Push `dev` → PR into `main` → babysit → migrate (if needed) → merge → Linear ship comments

Ship **this session’s finished work** by:

1. Identifying commits/changes that belong on local **`dev`**
2. **Refreshing from remote first:** `git fetch origin`, update local **`main`** from **`origin/main`**, then **merge `origin/main` into local `dev`** so `dev` has the latest trunk **before any push**
3. **Local Code Review Gate (Phase 1.5):** intensity-selected `grok-4.6` panel on `origin/main...dev` ([`../docs/intensity.md`](../docs/intensity.md)); if actionable findings exist, fix them on local `dev` from the merged scratch markdown via `prb-fixer` — **closed loop until clean** (or cycle cap / `--skip-review`). **One** Linear comment when the gate finishes. Then **runtime proof** per [`../docs/prove-it-works.md`](../docs/prove-it-works.md) — `--skip-review` skips the panel only, not the proof.
4. Pushing **`dev`** to **`origin/dev`** only after step 2 succeeds with a clean merge **and** step 3 is clean (or explicitly skipped)
5. Opening (or reusing) a PR: **base `main` ← head `dev`**
6. Babysitting every **5 minutes** for up to **15 minutes** total for CI + useful automated comments
7. **Production DB migrations** when the ship includes them: discover **this repo’s** migrate procedure and run it at the **pre-merge production gate** (see Phase 3.5 and [`references/db-migrations.md`](references/db-migrations.md))
8. **Merging the PR into `main`** only if the watch window ends with no useful automated feedback, CI is green (or never failed), and required production migrations have succeeded
9. **Linear ship comments** on every issue id in the ship set: PR-in-ship note at Phase 2; production note (PR + merge SHA + **deployment id/URL**) after merge (Phase 4.5) — see [`references/linear-ship-comments.md`](references/linear-ship-comments.md)

Default delivery **does** merge when the quiet window passes. Use `--no-merge` to stop after the watch without merging (and without applying production migrations unless the user explicitly asks).

The quiet babysit window and auto-merge logic start **only after** a clean local review (or `--skip-review`) + push + PR.

## Operating contract

- **Branches (fixed names, lowercase):** integration branch is **`dev`**; trunk is **`main`**. Never use capital-`D` `Dev`.
- **Always refresh trunk before push (hard rule):** **never** `git push origin dev` until you have (1) `git fetch origin`, (2) updated local **`main`** to match **`origin/main`** (ff-only when possible), and (3) **merged `origin/main` into local `dev`** with conflicts resolved. Pushing a stale `dev` that is missing latest `main` is a skill failure.
- **Local Code Review Gate before push (hard rule):** **never** `git push origin dev` and **never** open a PR until Phase 1.5 reports **zero actionable findings**, unless the user explicitly passed `--skip-review`. The gate is the intensity-selected `grok-4.6` panel in [`references/local-code-review.md`](references/local-code-review.md) (bands: [`../docs/intensity.md`](../docs/intensity.md); rubric: [`references/review-rubric.md`](references/review-rubric.md)). Orchestrator does not author findings.
- **Runtime proof before push (hard rule):** after the panel is clean (or skipped), follow [`../docs/prove-it-works.md`](../docs/prove-it-works.md) for in-scope ships. Matrix green is not a pass. `--skip-review` does **not** waive proof.
- **Closed-loop fixes on local `dev` only:** when the gate finds issues, merge findings into scratch markdown and spawn **one** `prb-fixer`. Do **not** file Linear issues or nested `/solve`. Do not push mid-loop. Cap full review→fix→re-review cycles (default **2**, override `--max-fix-cycles N`); if the cap is hit with remaining findings, **stop and report** — do not push. Post **one** `/prb — local review gate` comment on the first ship issue.
- **Session work only:** push commits that are already on local `dev` (or merge the session’s issue branch into local `dev` first if that is still the only place the work lives). Do not invent new features during `/prb` outside the review closed-loop fixes.
- **No force-push to `main`.** Prefer normal push to `dev`. If `dev` needs rewrite, use `--force-with-lease` only after a clear reason and never against `main`.
- **Never discard unrelated dirty files** (e.g. local hooks state, untracked scan dirs). Do not stage them.
- **Secrets:** never print Doppler/tokens/connection strings; never commit `.env`.
- **Babysit ≠ silent ignore:** every CI failure and every useful bot/human review comment is actionable. Auto-merge is forbidden while those exist.
- **Human veto:** if the user says stop/don’t merge in-session, cancel scheduled watches and do not merge.
- **Linear ship comments (hard rule when issues are known):** collect every Linear issue id in the ship set (commit messages, PR text). **Required:** comment on each with the PR URL when the PR is opened/reused (Phase 2). **Required after merge:** comment again with production ship evidence — PR, merge SHA, production **deployment id** + URL when discoverable (Vercel/`gh` deployments/project docs). Do **not** mark Done at PR open. After merge to `main`, mark ship-set issues **Done** (user asked 2026-08-13: `/prb` closes tickets). Do not steal foreign In Progress. `list_comments` immediately before every `save_comment` ([`../docs/linear-comments.md`](../docs/linear-comments.md)). Full procedure: [`references/linear-ship-comments.md`](references/linear-ship-comments.md). Phase 1.5 posts **one** gate comment (template C); it does not mint new Linear issues.
- **DB migrations follow the project (hard rule):** when the ship set includes schema/data migrations, discover and run **this repo’s** production migrate path from `AGENTS.md` / migration docs / `package.json` — do **not** invent Drizzle/Prisma/psql commands, Doppler project names, or configs. Prefer versioned `db:migrate` (or the repo’s documented equivalent). **Never** `db:push` / `drizzle-kit push` / `prisma db push` to production by default. **Never** print connection strings or Doppler secret values. **Never** auto-run content seeders as part of migrate. Full procedure: [`references/db-migrations.md`](references/db-migrations.md).
- **Migrate before merge (default):** if production migrations are required for the ship, apply them **after** the quiet window passes and **before** `gh pr merge`, so production deploy does not race ahead of schema (additive/expand path). Destructive migrations **block** auto-merge until the user explicitly approves.
- **Ship product build follows the project (hard rule):** when root `AGENTS.md` (or equivalent) documents a **required `/prb` ship product build** (e.g. rebuild + codesign a macOS `.app`), discover and run **that exact command** after Phase 1.5 is clean and **before** `git push origin dev`. Re-run after babysit fix pushes when the ship still touches app code. Failure blocks push. Do **not** invent archive/notary steps not documented. Full procedure: [`references/ship-product-build.md`](references/ship-product-build.md).

## Args

| Arg | Meaning |
|-----|---------|
| `--no-merge` | Watch only; never merge even if quiet; do not apply production migrations unless the user explicitly asks |
| `--skip-migrations` | Do not run production (or preview) migrate gates; ship may leave schema lagging — report a loud warning |
| `--skip-ship-build` | Do not run the project’s documented ship product build (e.g. macOS app rebuild); report a loud warning when AGENTS requires it |
| `--skip-review` | Skip the intensity-selected **panel** + closed-loop fix cycle (dangerous). **Does not** skip runtime proof. Loud warning in report |
| `--exhaustive-review` | After a clean first panel, run one more panel pass hunting for missed issues (default: **on** only when ship intensity is **critical**) |
| `--no-exhaustive` | One panel pass per cycle; still blocks on any actionable findings found |
| `--max-fix-cycles N` | Cap full review→fix→re-review cycles (default **2**) |
| `--watch-minutes N` | Total watch window (default **15**) |
| `--interval-minutes M` | Check cadence (default **5**) |

Parse these from the user message; ignore unknown tokens after logging them.

Record for the run: `SKIP_REVIEW`, `SHIP_INTENSITY`, `EXHAUSTIVE_REVIEW` (default from ship intensity unless flagged), `MAX_FIX_CYCLES` (default **2**), plus review exit fields from [`references/local-code-review.md`](references/local-code-review.md). Authority: [`../docs/intensity.md`](../docs/intensity.md).

---

## Phase 0 — Repo + session work inventory

1. Confirm git root; read `AGENTS.md` / `README.md` only if needed for push/PR conventions (Phase 1.5 will re-read AGENTS for Code Review Rules).
2. `gh auth status` — if not authenticated, stop.
3. Record:
   - `OWNER/REPO` via `gh repo view --json nameWithOwner --jq '.nameWithOwner'`
   - current branch, `git status -sb`
   - `git log origin/dev..dev --oneline` (if `origin/dev` exists)
   - `git log origin/main..dev --oneline` (work that would ship)
4. **Session work source of truth (in order):**
   1. Commits on local `dev` not on `origin/dev`
   2. Else commits on the current branch not in `dev` that were produced this session (merge them into local `dev` with a normal merge/ff after user-safe checks)
   3. Else if working tree has only intentional uncommitted session changes on an issue branch already meant for `dev`, stop and ask to commit via normal flow — **do not** auto-commit large unrelated dirt
5. If there is **nothing** to push (local `dev` == `origin/dev` and no new PR content vs `main`), report and exit.
6. **Migration inventory (always):** per [`references/db-migrations.md`](references/db-migrations.md) §1–2, record whether `origin/main...dev` changes migration/schema paths, and if so start the discovery table (`MIGRATE_CMD`, Doppler project/production config, forbidden push scripts, risk class). If migrations exist only uncommitted, stop and get them committed onto `dev` first.
7. **Ship product build inventory (always):** per [`references/ship-product-build.md`](references/ship-product-build.md), read `AGENTS.md` for a required `/prb` ship product build. Record `SHIP_BUILD_REQUIRED`, `SHIP_BUILD_CMD`, `SHIP_BUILD_OUTPUT`. If the user passed `--skip-ship-build`, note a loud skip for the report.
8. **Linear ship-id inventory (always):** per [`references/linear-ship-comments.md`](references/linear-ship-comments.md), start `SHIP_LINEAR_IDS` from `git log origin/main..dev --pretty=%B` (and branch names if useful). Re-scan before Phase 2 / 4.5 comments. Phase 1.5 does **not** add new Linear ids.

Unrelated dirty paths (leave alone): `.cursor/hooks/**`, `.deepsec/`, local env files, etc.

---

## Phase 1 — Refresh `origin/main` → local `dev` (no push)

**Hard gate:** Do **not** push anything to remote until local `dev` contains the latest `origin/main` **and** Phase 1.5 is clean (or `--skip-review`).

### 1A. Fetch and update local `main` from `origin/main`

```bash
git fetch origin

# Update local main to latest origin/main (ff-only preferred)
git checkout main
git merge --ff-only origin/main \
  || git pull --ff-only origin main
```

If ff-only fails (local `main` diverged), stop and report; do not rewrite `main` casually. Prefer reconciling with the user before shipping.

Confirm:

```bash
git rev-parse main
git rev-parse origin/main
# must be equal before continuing
```

### 1B. Check out local `dev` and merge latest main into it

```bash
# Prefer lowercase dev (migrate Dev → dev only if needed and safe; never delete Dev without asking)
git checkout dev 2>/dev/null \
  || git checkout -b dev origin/dev 2>/dev/null \
  || git checkout -b dev main

# REQUIRED: bring latest trunk into local dev BEFORE any push
git merge origin/main -m "Merge origin/main into dev before /prb push"
```

Rules for this merge:

- Always merge **`origin/main`** (not a stale local `main` pointer, though they should match after 1A).
- If merge conflicts: resolve them fully, run the repo’s relevant verify commands for touched files, commit the merge, then continue. **Do not push** until the merge is complete and clean **and** Phase 1.5 has passed.
- If the merge would destroy session work, stop and report; never drop session commits to “make the merge easy.”
- Optionally also merge `origin/dev` into local `dev` if remote dev is ahead and you need those commits: `git merge origin/dev` — still only after `origin/main` is in.

### 1C. Ensure session commits sit on `dev`

- Fast-forward or merge the session issue branch into `dev` if that work is not on `dev` yet.
- Re-verify `git log origin/main..dev --oneline` lists the intended ship set.

**Stop here.** Do **not** push yet. Proceed to Phase 1.5.

---

## Phase 1.5 — Local Code Review Gate + Closed-Loop Fix Cycle

**Authority:** [`references/local-code-review.md`](references/local-code-review.md).

**When:** after Phase 1 (1A–1C); **before** any `git push origin dev` and **before** Phase 2.

### Skip

If `--skip-review`: log a **loud** warning, set `REVIEW_EXIT=skipped`, skip 1.5A–1.5C (the panel). **Still run runtime proof** ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)) on in-scope ships; fail → do not push. Then Phase 1C½ then 1D. Do not invent this flag.

### 1.5A — Run Local Code Review

Follow [`references/local-code-review.md`](references/local-code-review.md) §3–6. Do not author findings in the orchestrator.

1. Compute the ship set (`git log` / three-dot diff / name-only on `origin/main...dev`).
2. Set `SHIP_INTENSITY` per [`../docs/intensity.md`](../docs/intensity.md) (`max` of stamps in the ship; bump to **critical** if the diff touches auth/billing/schema/migrations). Announce: `Ship intensity: <band> · panel <n> · exhaustive on|off`.
3. Load root + nested `AGENTS.md` **Code Review Rules** for changed paths.
4. Spawn the intensity-selected `grok-4.6` panel in one turn ([`references/local-code-review.md`](references/local-code-review.md) §6C) with [`references/review-rubric.md`](references/review-rubric.md) prepended and one overlay from [`references/reviewer-prompts.md`](references/reviewer-prompts.md). `subagent_type: prb-reviewer` (medium via the role). Do **not** pass a fake `effort:` field.
5. Merge JSON: dedupe, drop nits/speculation/pre-existing, cite rules when they apply. Actionable = P0/P1 plus high-signal P2 correctness/security/regression.
6. **Exhaustive:** default **on** only when `SHIP_INTENSITY` is **critical**. If the first panel is clean and exhaustive is on, run **one** more panel pass hunting for issues not already listed. `--exhaustive-review` / `--no-exhaustive` override.

**Nits are non-blocking.** Thoroughness-agent failure is fatal (do not push). Specialist failure: warn and continue.

**Gate pass (panel):** zero actionable findings → set `REVIEW_EXIT=clean` (or update after cycles).

**Then runtime proof (1.5D):** [`../docs/prove-it-works.md`](../docs/prove-it-works.md). Required when the ship set is in-scope (UI, auth, billing, public API, schema, shared helper). Fail → treat as actionable (re-enter the fixer loop or deny push). Only then Phase 1C½ then 1D.

### 1.5B — Closed-loop fix cycle (when actionable findings exist)

Authority: [`references/local-code-review.md`](references/local-code-review.md) §7.

```text
cycle = 0
while actionable findings remain:
  if cycle >= MAX_FIX_CYCLES:   # default 2
    STOP — do not push; do not open PR; report outstanding findings in the
    one gate comment + Phase 5
  cycle += 1

  Spawn one prb-fixer (model grok-4.6) with the merged cycle markdown.
  Fixer writes sibling …-fixes.md. Orchestrator commits product diff on
  local dev (never stage scratch), deletes that cycle’s scratch files.

  Re-run 1.5A on updated origin/main...dev
  Repeat until zero actionable findings
```

Rules:

- **No Linear tickets** for gate findings. Scratch markdown is the contract.
- All fixes land on **local `dev` only**. Never push inside the loop.
- After a fix cycle: stay on `dev`; if `origin/main` may have moved, re-fetch and re-merge Phase 1A–1B before re-review/push.
- If the fixer fails: leave findings open; do not pretend fixed; **deny push** if any actionable remain after re-review.
- Track `FINDINGS_FIXED_COUNT`, `REVIEW_CYCLES_USED`.
- When the gate exits (clean, cap, or tooling-block): **one** template C comment on the first `SHIP_LINEAR_ID` ([`references/linear-ship-comments.md`](references/linear-ship-comments.md)). No ship ids → report only.

### 1.5C — Safety limits

| Limit | Default |
|-------|---------|
| Full review→fix→re-review cycles | **2** (`--max-fix-cycles N`) |
| Panel passes per cycle | **1**, or **2** when exhaustive (default on **only** for critical ships) |
| Fixers per cycle | **one** (sequential extra only if paths conflict) |

On cycle cap with remaining findings: **stop**, report, **do not push**, **do not open PR**.

Only when the gate is **clean** (or skipped) may the skill proceed to Phase 1C½ (ship build when required) then Phase 1D.

---

## Phase 1C½ — Ship product build (when project requires it)

**Authority:** [`references/ship-product-build.md`](references/ship-product-build.md).

**When:** after Phase 1.5 is **clean** (or `--skip-review`); **before** Phase 1D `git push origin dev`.

### Skip

- Project has no documented ship product build → set `SHIP_BUILD=n/a`, continue to 1D.
- User passed `--skip-ship-build` → loud warning; set `SHIP_BUILD=skipped`; continue to 1D only if the user intentionally waived.

### Required (example: LeetBridge macOS app)

1. Confirm `SHIP_BUILD_CMD` from `AGENTS.md` (do not invent).
2. Run from the git root with a long timeout (Release + codesign).
3. On **non-zero exit**: **do not push**; report log tail; stop.
4. On success: record artifact path + timestamp; proceed to 1D.
5. After babysit **fix pushes**, re-run this phase before re-pushing when the ship still includes app/source paths (or whenever AGENTS says always rebuild).

Do **not** commit build outputs (`dist/`, archives) unless the user explicitly asks.

---

## Phase 1D — Push local `dev` to `origin/dev` (only after 1A–1C + clean 1.5 + ship build if required)

```bash
# Pre-push checklist (all must pass):
# - main == origin/main
# - dev contains origin/main (merge-base --is-ancestor origin/main dev)
# - working tree has no unresolved conflict markers from the main merge
# - Phase 1.5 CLEAN or --skip-review
# - Phase 1C½ ship product build succeeded or n/a or explicitly skipped
git merge-base --is-ancestor origin/main dev   # exit 0 required

git push -u origin dev
```

Rules:

- **Never** push to `origin/dev` if `origin/main` is not an ancestor of local `dev`.
- **Never** push if Phase 1.5 still has actionable findings (unless `--skip-review`).
- **Never** push if a required ship product build failed (unless `--skip-ship-build`).
- If push rejected non-ff, carefully merge `origin/dev` into local `dev` **without** dropping session commits, **re-run** `git merge origin/main` if main moved, **re-run Phase 1.5** if the ship set changed, **re-run Phase 1C½** if still required, then re-push. Use `--force-with-lease` only if the branch is exclusively this user’s integration branch and divergence is understood; prefer merge commit otherwise.
- After push, confirm: `git rev-parse dev` == `git rev-parse origin/dev`.

---

## Phase 2 — Open or reuse PR: `main` ← `dev`

**Prerequisite:** Phase 1D succeeded (push completed after clean review or skip).

```bash
# Existing open PR with head dev and base main?
gh pr list --state open --head dev --base main --json number,url,title
```

- **If exists:** reuse that PR number; do not open a duplicate.
- **If not:**

```bash
gh pr create --base main --head dev \
  --title "<concise title from commits / session>" \
  --body "$(cat <<'EOF'
## Summary
- <bullet list of what this ship includes, derived from commits / session>

## Test plan
- [ ] CI green on this PR
- [ ] /prb babysit window (default 15m @ 5m checks) observed no useful automated feedback
- [ ] Merge to main after quiet window (unless --no-merge)

## Notes
- Head: `dev` → Base: `main`
- Opened by `/prb`
- Local Code Review Gate: clean (intensity-selected grok-4.6 panel) | skipped (--skip-review)
EOF
)"
```

Title/body must be complete sentences and reflect **actual** commits (`git log origin/main..dev --oneline`).

If `MIGRATIONS_IN_SHIP=yes`, include a PR body bullet such as: pending production migrate via the repo’s documented command (will run at `/prb` pre-merge gate unless `--skip-migrations`).

Store `PR_NUMBER` and `PR_URL`.

### 2B — Linear “in ship” comments (required)

**Authority:** [`references/linear-ship-comments.md`](references/linear-ship-comments.md).

1. Refresh `SHIP_LINEAR_IDS` from ship commits + PR title/body.
2. For each id: **`list_comments` first**, then post template **A** only if that issue has no `/prb — in ship` comment for this `PR_NUMBER` (see [`references/linear-ship-comments.md`](references/linear-ship-comments.md)).
3. Record `LINEAR_PR_COMMENTS_POSTED` (includes skipped-as-already-present) and any failures. Continue the ship even if Linear is partially down — dump failed bodies into the final report.

Do **not** mark issues Done here. After **merge to `main`**, `/prb` **does** mark ship-set issues **Done** (see Phase 4.5). Claim protocol: `solve/references/multiplayer-linear.md`.

---

## Phase 3 — Babysit window (default 15 minutes, every 5 minutes)

### 3A. Define “useful comments / feedback”

Treat as **useful** (blocks auto-merge) if **any** of the following is true:

| Signal | Source |
|--------|--------|
| CI check `FAILURE` or `ERROR` | `gh pr checks` / `statusCheckRollup` |
| CI `CHANGES_REQUESTED` or failing required checks | PR review + branch protection |
| Unresolved review threads from bots or humans with actionable findings | GraphQL `reviewThreads` where `isResolved == false` and body is not pure emoji/LGTM |
| Top-level PR review state `CHANGES_REQUESTED` | `gh pr view --json reviewDecision` |
| Semgrep/CodeQL/Dependabot/Cursor/DeepSec/etc. comments that request a code change or report a defect | PR comments + review comments |
| Merge conflicts (`mergeable == CONFLICTING`) | `gh pr view` |

Treat as **not useful** (does **not** block merge by itself):

- Pure “LGTM”, emoji-only, “Looking good”
- Bot noise with no defect (e.g. “workflow started”, progress spinners)
- Pending/in-progress checks that have not failed yet (extend wait within the window; do not merge early while required checks are still pending if branch protection requires them)

### 3B. Check procedure (each tick)

At t=0 immediately after PR open/push, and every **interval** minutes until **watch-minutes** elapse:

1. `gh pr view $PR_NUMBER --json state,mergeable,mergeStateStatus,statusCheckRollup,reviewDecision,url`
2. `gh pr checks $PR_NUMBER` (or equivalent JSON)
3. Fetch unresolved review threads + recent issue comments; classify useful vs noise using §3A
4. Record tick result in a short session log (time, CI summary, useful-comment count)

**If useful feedback found mid-window:**

- **Do not merge**
- Fix in a worktree or on `dev` as appropriate (prefer worktree isolation for multi-file fixes if using subagents)
- Before re-pushing:
  1. `git fetch origin`, ensure local `main` matches `origin/main`, **merge `origin/main` into local `dev` again**
  2. **Re-run Phase 1.5** on the updated ship set (unless `--skip-review` for the whole run)
  3. **Re-run Phase 1C½** ship product build when the project requires it
  4. Only then `git push origin dev`
- **Reset or extend** the quiet clock: require a fresh quiet window of the full `watch-minutes` **or** at least one clean interval after the fix push—default: **restart the 15-minute quiet timer** from the fix push time
- Continue babysitting; never merge with open useful threads or red CI

**If CI still pending and no failures:** keep waiting until window ends or checks complete.

### 3C. Scheduling

Prefer durable scheduling so checks continue if the chat idles:

1. Create a **foreground or background** schedule only if the environment supports `scheduler_create` / `/loop`. Example intent:
   - interval: `5m`
   - prompt: continue `/prb` check cycle for PR `$PR_NUMBER` in repo `$OWNER/$REPO`; do not open a new PR; run Phase 3B once; if watch end reached, run Phase 3.5 then Phase 4. If the PR is **already MERGED**, do not merge again; for Phase 4.5, `list_comments` on each ship id and **skip** template B when `/prb — shipped to production` for this PR already exists. Mid-window ticks never post Linear ship comments.
2. Track `WATCH_STARTED_AT` (ISO UTC) and `WATCH_ENDS_AT = start + watch-minutes`.
3. Cap automatic ticks: `ceil(watch-minutes / interval-minutes)` plus the initial t=0 check (default: t=0, t=5, t=10, and final at t=15).
4. If schedulers are unavailable, run an explicit wait loop with `get_command_or_subagent_output` / shell sleep **only** if policy allows; otherwise perform t=0 check, tell the user to re-run `/prb check $PR_NUMBER` at 5m marks, and **do not merge** until a final check at ≥15m has been executed in-session.

Also support resume:

```text
/prb check [PR_NUMBER]
/prb merge [PR_NUMBER]   # only after quiet window + green CI; re-validate §3A + Phase 3.5 migrations; then Phase 4.5 Linear ship comments
```


### 3D. Optional deeper babysit

If CI fails or useful comments appear, you may reuse patterns from the `pr-babysit` skill (worktree fixes, reply to threads with commit SHAs). Cap automated code-fix attempts sensibly (e.g. 3 per cycle) and always re-merge main, **re-run Phase 1.5** when the ship set changed, then re-push `dev`.

### 3E. Preview DB (optional, only when migrations are in the ship)

If `MIGRATIONS_IN_SHIP=yes`, the project documents a **preview/stg** Doppler (or equivalent) config, and preview apps would break without schema: apply **preview** migrate once early in the window using the same discovered procedure with the preview config. Do not touch production here. Skip when `--skip-migrations` or when no preview database is documented.

If a fix push mid-window adds more migrations, re-inventory and re-apply preview migrate if you are using this step.

---

## Phase 3.5 — Production migration gate (before merge)

**When:** quiet window has passed (or final `/prb check` / `/prb merge` is about to merge), merge criteria below are otherwise satisfied, and you are **not** in `--no-merge` / `--skip-migrations`.

**Authority:** [`references/db-migrations.md`](references/db-migrations.md). Summary:

1. Re-confirm `MIGRATIONS_IN_SHIP` against the PR head SHA that will merge (`git diff --name-only origin/main...dev` with migration globs).
2. If **no** migrations in ship → skip; set report field `Migrations: none in ship`.
3. If migrations in ship:
   - Finish discovery table (script, Doppler project, **production** config name, forbidden commands).
   - Classify risk (additive / data / destructive).
   - **Destructive or unknown:** do **not** merge; stop for explicit user approval and ordering.
   - **Additive (default auto path):** run the project’s **production** migrate command successfully.
   - On migrate failure: **block merge**; report redacted error; do not retry blindly.
   - On success: proceed to Phase 4 merge immediately so deploy follows ready schema.
4. Prefer migrate **before** merge so production code does not deploy against missing schema. If project docs mandate a different order, follow the docs — but never leave required production schema unapplied after shipping dependent code.
5. Secrets: only via the project’s Doppler/`db:migrate` wrapper; never print URLs or tokens; never run seed/content loaders unless the user explicitly requests a named seed against a confirmed env.

`/prb merge` must re-run this gate (re-validate inventory + migrate if still pending) before merging.

---

## Phase 4 — End of window: merge or stop

At `WATCH_ENDS_AT` (or final `/prb check` after the window):

### Merge allowed only if **all** are true

1. PR still **open**
2. `mergeable == MERGEABLE` (not CONFLICTING)
3. No useful comments/threads per §3A
4. No failed/error CI checks; required checks completed successfully (or repo has no required checks and none failed)
5. `reviewDecision` is not `CHANGES_REQUESTED`
6. User did not pass `--no-merge` and did not veto
7. Head is still `dev` and base is still `main`
8. **Migrations:** either none in ship, or production migrate succeeded in Phase 3.5, or user passed `--skip-migrations` (warned), or user explicitly approved a documented alternate order that is already complete

### Merge command

Prefer GitHub merge (respect repo settings):

```bash
gh pr merge $PR_NUMBER --merge   # or --squash if repo standard is squash; prefer repo default
# If admin rights needed and policy allows: only with explicit user consent
```

After merge:

```bash
git fetch origin
git checkout main
git merge --ff-only origin/main || git reset --hard origin/main  # only if local main has no unique work
git checkout dev
git merge origin/main -m "Merge main into dev after /prb ship"
git push origin dev   # keep origin/dev ≥ main
```

Do **not** `reset --hard` if it would destroy unique local commits.

If Phase 3.5 was skipped incorrectly and production code now requires unapplied schema, **run production migrate immediately** using the discovered project command, then report the incident — do not leave production broken.

**After a successful merge:** immediately run **Phase 4.5** (Linear production ship comments) before the user report.

### If merge blocked

Report clearly:

- PR URL
- Why blocked (failed check names, comment excerpts, conflicts, **failed/pending production migrate**, destructive migration awaiting approval)
- Next action (`/prb check`, fix + Phase 1.5 + push `dev`, complete migrate, or human merge)
- Whether Phase 2 Linear PR comments already landed (`LINEAR_PR_COMMENTS_POSTED`)

Cancel any scheduled `/prb` ticks for this PR when terminal (merged or abandoned). Do **not** post production ship comments if merge did not happen.

---

## Phase 4.5 — Linear production ship comments (after merge)

After a successful merge to `main`, for each id in `SHIP_LINEAR_IDS` that this ship implemented: set state **Done** (in addition to the production comment). Skip ids that are still In Progress with a **foreign** live `claimed-by:` comment.

**When:** PR was **merged** to `main` in Phase 4 (or a resume path confirms the PR is already merged and production comments were not posted yet).  
**Skip:** `--no-merge` and merge never happened; or `SHIP_LINEAR_IDS` is empty after a full rescan.

**Authority:** [`references/linear-ship-comments.md`](references/linear-ship-comments.md).

1. Resolve `MERGE_SHA` on `main` (`git rev-parse origin/main` after fetch, or PR merge commit from `gh pr view`).
2. Re-scan issue ids from the merged range if needed.
3. **Discover production deployment** for `MERGE_SHA` (poll up to ~2–3 minutes):
   - GitHub deployments/statuses for `production`
   - Vercel CLI / MCP / project docs (`dpl_…`, inspect URL, production URL)
   - Other host documented in `AGENTS.md`
   - If none: `DEPLOY_ID=n/a` with reason (library repo / still pending / no CD)
4. For each id in `SHIP_LINEAR_IDS`: **`list_comments` first**. Post template **B** only if there is no `/prb — shipped to production` comment for this `PR_NUMBER` or `MERGE_SHA`. If one already exists, skip (do not post a second). Template B fields:
   - PR number + URL
   - Merge commit SHA
   - Production **deployment ID**
   - Production deployment URL / inspector link
   - Migrate note (config **name** only, no secrets)
   - ISO UTC timestamp
5. If the issue is already **Done** and template B is present, leave status alone. Otherwise set **Done** after the comment (or skip) as in the operating contract. Skip **Done** on ids that are still In Progress with a **foreign** live `claimed-by:`.
6. Record `LINEAR_SHIP_COMMENTS_POSTED` (includes skipped-as-already-present), `DEPLOY_ID`, `DEPLOY_URL`, failures.

If Linear fails entirely: include the full comment body once in Phase 5 under **Linear comments (not posted)** plus the id list.

---

## Phase 5 — User report

```markdown
** /prb complete**
**Repo:** owner/repo
**Review:** ship intensity <band> · panel clean after N cycles (roles…, grok-4.6) | local fixed M findings from scratch | skipped (--skip-review) | blocked at cycle cap (remaining …) | blocked (fixer/thoroughness agent failed)
**Runtime proof:** driven `<path>` → `<observed>` | n/a (out of scope) | blocked (unproven)
**Gate comment:** `/prb — local review gate` on TEAM-123 | none (no ship ids) | failed
**Linear ship set:** TEAM-123, TEAM-124, … | none detected
**Linear ship comments:** PR notes on K issues · production notes on K issues · failures: none | <ids>
**Deploy:** id `dpl_…` | url <…> | pending | n/a (<reason>)
**Ship build:** n/a | rebuilt `<path>` @ <time> | skipped (--skip-ship-build) | blocked (<reason>)
**Pushed:** origin/dev @ <sha> | not pushed (<reason>)
**PR:** #N — <url> (base main ← head dev) | not opened (<reason>)
**Watch:** <minutes>m @ <interval>m checks | n/a
**Useful feedback:** none | <summary>
**Migrations:** none in ship | applied production (`<config>`) @ <time> | blocked (<reason>) | skipped (--skip-migrations|--no-merge)
**Migrate:** <stack/command shape, no secrets> | n/a
**Risk class:** additive | data | destructive | n/a
**Merge:** merged @ <sha> | skipped (<reason>) | blocked (<reason>)
**Local:** main/dev synced notes
```

If Phase 1.5 stopped the ship before push, still emit this report with `Pushed: not pushed`, `PR: not opened`, and full Review / Linear lines so the user can continue manually.

---

## Safety guardrails

- **Never push to `origin/dev` until `origin/main` is merged into local `dev`** (verify with `git merge-base --is-ancestor origin/main dev`)
- **Never push to `origin/dev` or open a PR until Phase 1.5 is clean** (zero actionable findings), unless `--skip-review` was explicit
- **Never push while the closed-loop fix cycle still has open actionable findings** or while under `blocked-at-cap` / `blocked-tooling`
- Always `git fetch origin` and ff-update local `main` from `origin/main` before that merge
- Never merge on red CI or with unresolved useful review threads
- Never force-push `main`
- Never commit secrets or unrelated dirty files
- Never open a second duplicate `main`←`dev` PR if one is already open
- Never auto-merge with `--no-merge`
- If branch protection forbids the agent merge, report the exact `gh` error and stop (do not bypass without explicit user request)
- **Never merge a ship that includes DB migrations without completing Phase 3.5** (unless `--skip-migrations` or explicit user waiver)
- **Never invent** migrate commands or Doppler production config names; follow the project under the current git root
- **Never** `db:push` / schema push to production by default; never print `DATABASE_URL` or Doppler secrets; never auto-seed CMS/content as part of `/prb`
- Do not file Linear issues for gate findings; all closed-loop fixes land on local `dev` only
- **Never skip the intensity-selected `grok-4.6` panel** in Phase 1.5 (unless `--skip-review`); orchestrator does not substitute its own review. Do not run a 4-agent panel on a light ship, and do not skip security on a critical ship.
- **Never skip runtime proof** on an in-scope ship ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)); `--skip-review` is not a waiver
- **Never push if the thoroughness agent failed** to return valid JSON, even if specialists were clean
- **Never push when a required ship product build failed** (unless `--skip-ship-build`); always re-run the documented build after babysit fixes that change app sources when AGENTS requires it
- **Never skip Linear ship comments** when `SHIP_LINEAR_IDS` is non-empty and the PR was opened (Phase 2) or merged (Phase 4.5) — post what you know even if deploy id is still pending
- **Never put secrets** (Doppler values, connection strings, tokens) in Linear ship comments

## Anti-patterns

- Pushing `dev` without first pulling/merging latest `origin/main` into local `dev`
- Pushing `dev` or opening a PR without a clean Local Code Review Gate (unless `--skip-review`)
- Skipping re-review after fixer commits
- Orchestrator-authored findings instead of the intensity-selected panel
- Reviewing Phase 1.5 on a model other than `grok-4.6` (or omitting `model` so Cursor/Claude inherit)
- Treating nits as ship-blockers — or treating P0/P1 findings as optional nits
- Treating a clean review panel or typecheck as runtime proof for UI/auth/billing/API/schema/shared-helper ships
- Filing Linear issues or nested `/solve` for gate findings
- Fixing review findings only on a remote branch / PR without landing on local `dev` first
- Staging or committing `$TMPDIR` review scratch
- Infinite review→fix loops without honoring `--max-fix-cycles`
- Merging a stale local `main` that was never ff’d to `origin/main`
- Pushing an issue branch as `dev` by renaming without merging history
- Merging at t=0 without watching when the user asked for the default babysit window
- Treating “pending CI” as “green” and merging early under required checks
- Closing the PR instead of merging when the window is quiet
- Merging code that needs new tables/columns without running **this project’s** production migrate
- Running `drizzle-kit push` / `db:push` against production “because migrate was slow”
- Using another repo’s Doppler project/config or a hardcoded migrate one-liner that is not in this repo’s docs/scripts
- Auto-running destructive `DROP`/`RENAME` migrations without explicit user approval
- Printing connection strings while “verifying” migrate readiness
- Re-pushing babysit fixes without re-running Phase 1.5 when the ship set changed
- Shipping a native/macOS app repo that documents a required ship build without running it before push
- Inventing notary/archive steps that `AGENTS.md` does not document
- Merging to production without commenting on ship Linear issues (PR + deploy id when available)
- Posting a Linear ship comment without `list_comments` first
- Spamming duplicate `/prb` ship comments (orchestrator racing a babysit tick, or re-run on the same PR)
- Commenting production “shipped” on issues when the PR never merged
- Auto-closing or reopening Linear issues as a substitute for a clear ship comment
- Spawning a 4-agent panel on a light ship, or skipping security on a critical ship
- Passing a fake `effort:` field on `spawn_subagent` (use `prb-reviewer` / `prb-fixer`)
- Running exhaustive review on a non-critical ship without `--exhaustive-review`

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/issue` | Files thorough Linear tickets only; **not** used in Phase 1.5 |
| `/solve` | Cheap construction onto **local** `dev`; **not** the `/prb` closed loop |
| `/review` | Optional local/branch/PR review tooling; **not** the `/prb` ship gate (Phase 1.5 is local-only and does not post GitHub PENDING reviews) |
| `/pr-babysit` | Watches arbitrary PR numbers; does not define the push-`dev`/open-`main` flow |
| `/prb` | End-to-end: session → **intensity-selected grok-4.6 review gate + scratch fixer** → `origin/dev` → PR into `main` → timed babysit → **project production migrate when needed** → merge → **Linear ship comments (PR + deploy id)** |
