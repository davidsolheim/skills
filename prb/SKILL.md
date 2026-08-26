---
name: prb
description: >
  Ship session work: fetch origin/main into local main, merge origin/main into
  local dev, run a thorough multi-agent Local Code Review Gate on grok-4.6
  (with closed-loop Linear /issue + /solve fixes on local dev until clean),
  then push to origin/dev; open a PR from dev into main; babysit CI/bot feedback on a
  5-minute cadence for up to 15 minutes, push to **origin/dev** for user review (no approval needed for `dev`), then **stop and wait for explicit user approval**
  before merging to **main** or running production migrate/deploy (quiet CI alone is
  never main/prod authority). When the ship includes DB
  migrations, discover and run this project's production migration procedure at
  the correct pre-merge gate (never invent a stack; never db:push to prod by
  default). Use when the user runs /prb, says "push dev and PR to main", "ship
  session work", "babysit then merge to main", "promote dev to main with CI
  watch", or production migrate-on-ship.
argument-hint: "[--no-merge] [--skip-migrations] [--skip-review] [--exhaustive-review|--no-exhaustive] [--max-fix-cycles N] [--watch-minutes N] [--interval-minutes M]"
---

# /prb — Push `dev` → PR into `main` → babysit → migrate (if needed) → merge

Ship **this session’s finished work** by:

1. Identifying commits/changes that belong on local **`dev`**
2. **Refreshing from remote first:** `git fetch origin`, update local **`main`** from **`origin/main`**, then **merge `origin/main` into local `dev`** so `dev` has the latest trunk **before any push**
3. **Local Code Review Gate (Phase 1.5):** four parallel `grok-4.6` reviewers (thoroughness, security, rules, challenge) on `origin/main...dev`; if actionable findings exist, create Linear issues via `/issue`, fix them on local `dev` via `/solve`, re-run the panel — **closed loop until clean** (or cycle cap / `--skip-review`). Then **runtime proof** per [`../docs/prove-it-works.md`](../docs/prove-it-works.md) — `--skip-review` skips the panel only, not the proof.
4. Pushing **`dev`** to **`origin/dev`** only after step 2 succeeds with a clean merge **and** step 3 is clean (or explicitly skipped)
5. Opening (or reusing) a PR: **base `main` ← head `dev`**
6. Babysitting every **5 minutes** for up to **15 minutes** total for CI + useful automated comments
7. **Production DB migrations** when the ship includes them: discover **this repo’s** migrate procedure and run it at the **pre-merge production gate** (see Phase 3.5 and [`references/db-migrations.md`](references/db-migrations.md))
8. **Reporting merge-ready** after a quiet green window — then **merging the PR into `main` only after explicit user approval** (and required production migrations when approved)

Default delivery **does** push **`origin/dev`** (after checks) and open/babysit the PR, but **does not** merge to `main`. After a clean quiet window, report **merge-ready for production** and **wait for explicit user approval** (e.g. "merge it", "approve merge", `/prb merge` after they said yes). Quiet CI is **never** merge authority. `--no-merge` is the default posture (redundant but allowed). Never merge to `main` or run production migrate/deploy without that approval.

The quiet babysit window proves readiness only; it does **not** authorize merge. Merge/prod ship requires explicit user approval after the window.

## Operating contract

- **Branches (fixed names, lowercase):** integration branch is **`dev`**; trunk is **`main`**. Never use capital-`D` `Dev`.
- **Long-lived `dev` on every project (hard):** local + `origin/dev` must exist. If `origin/dev` is missing, create it from current `main` (`git push -u origin dev`) before shipping. Always refresh: `fetch` → local `main` = `origin/main` → **merge `origin/main` into local `dev`** → build/verify/security on the ship set → only then `git push origin dev`.
- **`origin/dev` does not need approval:** after Phase 1.5 clean + build/security gates, push to `origin/dev` automatically so the user can review there.
- **`main`/production always need explicit approval:** never merge the PR to `main` or run production deploy/migrate until the user clearly says so in-session.
- **Always refresh trunk before push (hard rule):** **never** `git push origin dev` until you have (1) `git fetch origin`, (2) updated local **`main`** to match **`origin/main`** (ff-only when possible), and (3) **merged `origin/main` into local `dev`** with conflicts resolved. Pushing a stale `dev` that is missing latest `main` is a skill failure.
- **Local Code Review Gate before push (hard rule):** **never** `git push origin dev` and **never** open a PR until Phase 1.5 reports **zero actionable findings**, unless the user explicitly passed `--skip-review`. The gate is the four-agent `grok-4.6` panel in [`references/local-code-review.md`](references/local-code-review.md) (rubric: [`references/review-rubric.md`](references/review-rubric.md)). Orchestrator does not author findings.
- **Runtime proof before push (hard rule):** after the panel is clean (or skipped), follow [`../docs/prove-it-works.md`](../docs/prove-it-works.md) for in-scope ships. Matrix green is not a pass. `--skip-review` does **not** waive proof.
- **Closed-loop fixes on local `dev` only:** when the gate finds issues, file Linear tickets via `/issue` and implement via `/solve` onto **local `dev`**. Do not push mid-loop. Prefer one Linear issue per distinct problem. Cap full review→issue→solve cycles (default **5**, override `--max-fix-cycles N`); if the cap is hit with remaining findings, **stop and report** — do not push.
- **Session work only:** push commits that are already on local `dev` (or merge the session’s issue branch into local `dev` first if that is still the only place the work lives). Do not invent new features during `/prb` outside the review closed-loop fixes.
- **No force-push to `main`.** Prefer normal push to `dev`. If `dev` needs rewrite, use `--force-with-lease` only after a clear reason and never against `main`.
- **Never discard unrelated dirty files** (e.g. local hooks state, untracked scan dirs). Do not stage them.
- **Secrets:** never print Doppler/tokens/connection strings; never commit `.env`.
- **Babysit ≠ silent ignore:** every CI failure and every useful bot/human review comment is actionable. Merge is forbidden while those exist.
- **Explicit approval required for main/prod (hard rule):** never `gh pr merge`, never merge to `main`, and never run production migrate/deploy solely because the quiet window passed or CI is green. Only proceed when the user **explicitly** approves in this session (e.g. "merge PR N", "approve merge", or `/prb merge` after they said yes). Quiet babysit = readiness report, not authority.
- **Human veto:** if the user says stop/don’t merge in-session, cancel scheduled watches and do not merge.
- **Linear:** if commits mention issue IDs (e.g. `TEAM-123`), add the PR URL as a comment on those issues when practical. Do not invent Linear state changes at PR-open beyond optional **In Review**. After **explicit merge to `main`**, mark ship-set issues **Done** and post the production ship comment. Do not steal or close foreign In Progress. See `../solve/references/multiplayer-linear.md`. Phase 1.5 may create and solve additional Linear issues for review findings — track those IDs in the final report.
- **DB migrations follow the project (hard rule):** when the ship set includes schema/data migrations, discover and run **this repo’s** production migrate path from `AGENTS.md` / migration docs / `package.json` — do **not** invent Drizzle/Prisma/psql commands, Doppler project names, or configs. Prefer versioned `db:migrate` (or the repo’s documented equivalent). **Never** `db:push` / `drizzle-kit push` / `prisma db push` to production by default. **Never** print connection strings or Doppler secret values. **Never** auto-run content seeders as part of migrate. Full procedure: [`references/db-migrations.md`](references/db-migrations.md).
- **Migrate before merge (default):** if production migrations are required for the ship, apply them **after** the quiet window passes and **before** `gh pr merge`, so production deploy does not race ahead of schema (additive/expand path). Destructive migrations **block** auto-progress and still require explicit user approval before merging to main/prod.

## Args

| Arg | Meaning |
|-----|---------|
| `--no-merge` | Default posture (redundant): watch only; never merge; do not apply production migrations unless the user explicitly asks |
| *(explicit approval)* | Required before any merge to `main` or production migrate/deploy — user must say so in-session |
| `--skip-migrations` | Do not run production (or preview) migrate gates; ship may leave schema lagging — report a loud warning |
| `--skip-review` | Skip the four-agent **panel** + closed-loop fix cycle (dangerous). **Does not** skip runtime proof. Loud warning in report |
| `--exhaustive-review` | After a clean first panel, run one more panel pass for missed issues (default already **on**) |
| `--no-exhaustive` | One panel pass per cycle; still blocks on any actionable findings found |
| `--max-fix-cycles N` | Cap full review→issue→solve→re-review cycles (default **5**) |
| `--watch-minutes N` | Total watch window (default **15**) |
| `--interval-minutes M` | Check cadence (default **5**) |

Parse these from the user message; ignore unknown tokens after logging them.

Record for the run: `SKIP_REVIEW`, `EXHAUSTIVE_REVIEW` (default true), `MAX_FIX_CYCLES` (default 5), plus review exit fields from [`references/local-code-review.md`](references/local-code-review.md).

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

If `--skip-review`: log a **loud** warning, set `REVIEW_EXIT=skipped`, skip 1.5A–1.5C (the panel). **Still run runtime proof** ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)) on in-scope ships; fail → do not push. Then Phase 1D. Do not invent this flag.

### 1.5A — Run Local Code Review

Follow [`references/local-code-review.md`](references/local-code-review.md) §3–6. Do not author findings in the orchestrator.

1. Compute the ship set (`git log` / three-dot diff / name-only on `origin/main...dev`).
2. Load root + nested `AGENTS.md` **Code Review Rules** for changed paths.
3. Spawn **four** `grok-4.6` reviewers in one turn (thoroughness, security, rules, challenge) with [`references/review-rubric.md`](references/review-rubric.md) prepended and one overlay from [`references/reviewer-prompts.md`](references/reviewer-prompts.md).
4. Merge JSON: dedupe, drop nits/speculation/pre-existing, cite rules when they apply. Actionable = P0/P1 plus high-signal P2 correctness/security/regression.
5. **Exhaustive (default on):** if the first panel is clean, run **one** more panel pass hunting for issues not already listed. `--no-exhaustive` = one panel.

**Nits are non-blocking.** Thoroughness-agent failure is fatal (do not push). Specialist failure: warn and continue.

**Gate pass (panel):** zero actionable findings → set `REVIEW_EXIT=clean` (or update after cycles).

**Then runtime proof:** [`../docs/prove-it-works.md`](../docs/prove-it-works.md). Required when the ship set is in-scope (UI, auth, billing, public API, schema, shared helper). Fail → treat as actionable (file `/issue` + `/solve` or deny push). Only then Phase 1D.

### 1.5B — Closed-loop fix cycle (when actionable findings exist)

```text
cycle = 0
while actionable findings remain:
  if cycle >= MAX_FIX_CYCLES:   # default 5
    STOP — do not push; do not open PR; report outstanding findings + Linear ids
  cycle += 1

  For each distinct actionable finding (or tightly related group):
    1. Spawn subagent → /issue  (file/line, severity, AGENTS rule, context; model grok-4.6)
    2. Capture TEAM-### 
    3. Spawn subagent → /solve TEAM-###  (fix lands on local dev only; no push; model grok-4.6)
       Prefer sequential solves so merges to dev do not race.

  Re-run 1.5A (full four-agent panel) on updated origin/main...dev
  Repeat until zero actionable findings
```

Rules:

- **One Linear issue per distinct problem** — not one giant issue for unrelated defects.
- All fixes merge to **local `dev` only**. Never push inside the loop.
- After solves: stay on `dev`; if `origin/main` may have moved, re-fetch and re-merge Phase 1A–1B before re-review/push.
- If `/issue` fails (Linear down): `REVIEW_EXIT=blocked-tooling`; report drafts; **do not push**.
- If `/solve` fails: do not pretend fixed; deny push while actionable remain after re-review.
- Track `LINEAR_ISSUES_CREATED`, `LINEAR_ISSUES_SOLVED`, `FINDINGS_FIXED_COUNT`, `REVIEW_CYCLES_USED`.

### 1.5C — Safety limits

| Limit | Default |
|-------|---------|
| Full review→issue→solve cycles | **5** (`--max-fix-cycles N`) |
| Panel passes per cycle | **1**, or **2** when exhaustive (default on) |
| Issues per problem | one distinct issue (tight groups only) |

On cycle cap with remaining findings: **stop**, report, **do not push**, **do not open PR**.

Only when the gate is **clean** (or skipped) may the skill proceed to Phase 1D.

---

## Phase 1D — Push local `dev` to `origin/dev` (only after 1A–1C + clean 1.5)

```bash
# Pre-push checklist (all must pass):
# - main == origin/main
# - dev contains origin/main (merge-base --is-ancestor origin/main dev)
# - working tree has no unresolved conflict markers from the main merge
# - Phase 1.5 CLEAN or --skip-review
git merge-base --is-ancestor origin/main dev   # exit 0 required

git push -u origin dev
```

Rules:

- **Never** push to `origin/dev` if `origin/main` is not an ancestor of local `dev`.
- **Never** push if Phase 1.5 still has actionable findings (unless `--skip-review`).
- If push rejected non-ff, carefully merge `origin/dev` into local `dev` **without** dropping session commits, **re-run** `git merge origin/main` if main moved, **re-run Phase 1.5** if the ship set changed, then re-push. Use `--force-with-lease` only if the branch is exclusively this user’s integration branch and divergence is understood; prefer merge commit otherwise.
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
- Local Code Review Gate: clean (4-agent grok-4.6 panel) | skipped (--skip-review)
EOF
)"
```

Title/body must be complete sentences and reflect **actual** commits (`git log origin/main..dev --oneline`).

If `MIGRATIONS_IN_SHIP=yes`, include a PR body bullet such as: pending production migrate via the repo’s documented command (will run at `/prb` pre-merge gate unless `--skip-migrations`).

If Phase 1.5 created Linear issues, optionally list them in the PR body under Notes.

Store `PR_NUMBER` and `PR_URL`.

Optional: comment on related Linear issues with `PR_URL` (session commit issues + Phase 1.5 issues when practical).

---

## Phase 3 — Babysit window (default 15 minutes, every 5 minutes)

### 3A. Define “useful comments / feedback”

Treat as **useful** (blocks merge — and you must still get explicit user approval before merging to main/prod).

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
  3. Only then `git push origin dev`
- **Reset or extend** the quiet clock: require a fresh quiet window of the full `watch-minutes` **or** at least one clean interval after the fix push—default: **restart the 15-minute quiet timer** from the fix push time
- Continue babysitting; never merge with open useful threads or red CI

**If CI still pending and no failures:** keep waiting until window ends or checks complete.

### 3C. Scheduling

Prefer durable scheduling so checks continue if the chat idles:

1. Create a **foreground or background** schedule only if the environment supports `scheduler_create` / `/loop`. Example intent:
   - interval: `5m`
   - prompt: continue `/prb` check cycle for PR `$PR_NUMBER` in repo `$OWNER/$REPO`; do not open a new PR; run Phase 3B once; if watch end reached, run Phase 3.5 (project production migrations when in ship) then Phase 4
2. Track `WATCH_STARTED_AT` (ISO UTC) and `WATCH_ENDS_AT = start + watch-minutes`.
3. Cap automatic ticks: `ceil(watch-minutes / interval-minutes)` plus the initial t=0 check (default: t=0, t=5, t=10, and final at t=15).
4. If schedulers are unavailable, run an explicit wait loop with `get_command_or_subagent_output` / shell sleep **only** if policy allows; otherwise perform t=0 check, tell the user to re-run `/prb check $PR_NUMBER` at 5m marks, and **do not merge** until a final check at ≥15m has been executed in-session.

Also support resume:

```text
/prb check [PR_NUMBER]
/prb merge [PR_NUMBER]   # only after quiet window + green CI; re-validate §3A + Phase 3.5 migrations
```

### 3D. Optional deeper babysit

If CI fails or useful comments appear, you may reuse patterns from the `pr-babysit` skill (worktree fixes, reply to threads with commit SHAs). Cap automated code-fix attempts sensibly (e.g. 3 per cycle) and always re-merge main, **re-run Phase 1.5** when the ship set changed, then re-push `dev`.

### 3E. Preview DB (optional, only when migrations are in the ship)

If `MIGRATIONS_IN_SHIP=yes`, the project documents a **preview/stg** Doppler (or equivalent) config, and preview apps would break without schema: apply **preview** migrate once early in the window using the same discovered procedure with the preview config. Do not touch production here. Skip when `--skip-migrations` or when no preview database is documented.

If a fix push mid-window adds more migrations, re-inventory and re-apply preview migrate if you are using this step.

---

## Phase 3.5 — Production migration gate (before merge)

**When:** quiet window has passed, the user has **explicitly approved** merge/production ship in this session, merge criteria are otherwise satisfied, and you are **not** in `--no-merge` / `--skip-migrations`. Without explicit approval, skip this phase and report migrations as pending approval.

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

### Merge is never automatic

At end of window, if the PR looks green/quiet: **do not merge**. Report merge-ready status and **ask for explicit user approval**. Only after the user clearly approves in this session may you run the merge checklist below.

### Merge allowed only if **all** are true (and user explicitly approved)

1. PR still **open**
2. `mergeable == MERGEABLE` (not CONFLICTING)
3. No useful comments/threads per §3A
4. No failed/error CI checks; required checks completed successfully (or repo has no required checks and none failed)
5. `reviewDecision` is not `CHANGES_REQUESTED`
6. User **explicitly approved** merge in this session (not merely silent / not merely green CI). `--no-merge` or any veto blocks merge
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

### If merge blocked

Report clearly:

- PR URL
- Why blocked (failed check names, comment excerpts, conflicts, **failed/pending production migrate**, destructive migration awaiting approval)
- Next action (`/prb check`, fix + Phase 1.5 + push `dev`, complete migrate, or human merge)

Cancel any scheduled `/prb` ticks for this PR when terminal (merged or abandoned). Do **not** post production ship comments or mark Done if merge did not happen.

---

## Phase 4.5 — Linear production closeout (after merge)

After a successful merge to `main`, for each Linear id in the ship set that this ship implemented:

1. Comment: PR URL, merge SHA, production deploy id/URL when known.
2. Set state **Done**.
3. Skip ids that are still In Progress with a **foreign** live `claimed-by:` comment (`../solve/references/multiplayer-linear.md`).

Do **not** mark Done at PR open (Phase 2 comments only).

---

## Phase 5 — User report

```markdown
** /prb complete**
**Repo:** owner/repo
**Review:** panel clean after N cycles (thoroughness/security/rules/challenge, grok-4.6) | local fixed M findings across K Linear issues | skipped (--skip-review) | blocked at cycle cap (remaining …) | blocked (Linear/solve failure | thoroughness agent failed)
**Runtime proof:** driven `<path>` → `<observed>` | n/a (out of scope) | blocked (unproven)
**Linear issues created & solved:** TEAM-123, TEAM-124, … | none
**Pushed:** origin/dev @ <sha> | not pushed (<reason>)
**PR:** #N — <url> (base main ← head dev) | not opened (<reason>)
**Watch:** <minutes>m @ <interval>m checks | n/a
**Useful feedback:** none | <summary>
**Migrations:** none in ship | applied production (`<config>`) @ <time> | blocked (<reason>) | skipped (--skip-migrations|--no-merge)
**Migrate:** <stack/command shape, no secrets> | n/a
**Risk class:** additive | data | destructive | n/a
**Merge:** awaiting explicit approval | merged @ <sha> (user approved) | skipped (<reason>) | blocked (<reason>)
**Linear:** PR comments | Done after merge (<ids>) | skipped foreign claims
**Local:** main/dev synced notes
```

If Phase 1.5 stopped the ship before push, still emit this report with `Pushed: not pushed`, `PR: not opened`, and full Review / Linear lines so the user can continue manually.

---

## Safety guardrails

- **Never push to `origin/dev` until `origin/main` is merged into local `dev`** (verify with `git merge-base --is-ancestor origin/main dev`)
- **Never push to `origin/dev` or open a PR until Phase 1.5 is clean** (zero actionable findings), unless `--skip-review` was explicit
- **Never push while the closed-loop fix cycle still has open actionable findings** or while under `blocked-at-cap` / `blocked-tooling`
- Always `git fetch origin` and ff-update local `main` from `origin/main` before that merge
- Never merge to main/prod without explicit user approval
- Never merge on red CI or with unresolved useful review threads
- Never force-push `main`
- Never commit secrets or unrelated dirty files
- Never open a second duplicate `main`←`dev` PR if one is already open
- Never merge to main/prod without explicit user approval
- If branch protection forbids the agent merge, report the exact `gh` error and stop (do not bypass without explicit user request)
- **Never merge a ship that includes DB migrations without completing Phase 3.5** (unless `--skip-migrations` or explicit user waiver)
- **Never invent** migrate commands or Doppler production config names; follow the project under the current git root
- **Never** `db:push` / schema push to production by default; never print `DATABASE_URL` or Doppler secrets; never auto-seed CMS/content as part of `/prb`
- Prefer one Linear issue per distinct review finding; all closed-loop fixes land on local `dev` only
- **Never skip the four-agent `grok-4.6` panel** in Phase 1.5 (unless `--skip-review`); orchestrator does not substitute its own review
- **Never skip runtime proof** on an in-scope ship ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)); `--skip-review` is not a waiver
- **Never push if the thoroughness agent failed** to return valid JSON, even if specialists were clean

## Anti-patterns

- Pushing `dev` without first pulling/merging latest `origin/main` into local `dev`
- Pushing `dev` or opening a PR without a clean Local Code Review Gate (unless `--skip-review`)
- Skipping re-review after `/solve` closed-loop fixes
- Orchestrator-authored findings instead of the thoroughness/security/rules/challenge panel
- Reviewing Phase 1.5 on a model other than `grok-4.6` (or omitting `model` so Cursor/Claude inherit)
- Treating nits as ship-blockers — or treating P0/P1 findings as optional nits
- Treating a clean review panel or typecheck as runtime proof for UI/auth/billing/API/schema/shared-helper ships
- Bundling unrelated review findings into one mega Linear issue
- Fixing review findings only on a remote branch / PR without landing on local `dev` first
- Infinite review→fix loops without honoring `--max-fix-cycles`
- Merging a stale local `main` that was never ff’d to `origin/main`
- Pushing an issue branch as `dev` by renaming without merging history
- Auto-merging to main/prod without explicit user approval (quiet CI is not approval)
- Requiring user approval before `git push origin dev` after green checks (dev push is automatic)
- Pushing `origin/dev` before build/security/verify checks pass
- Working in a repo that lacks long-lived local + `origin` `dev` without creating it
- Merging at t=0 without watching when the user asked for the default babysit window
- Treating “pending CI” as “green” and merging early under required checks
- Closing the PR instead of merging when the window is quiet
- Merging code that needs new tables/columns without running **this project’s** production migrate
- Running `drizzle-kit push` / `db:push` against production “because migrate was slow”
- Using another repo’s Doppler project/config or a hardcoded migrate one-liner that is not in this repo’s docs/scripts
- Auto-running destructive `DROP`/`RENAME` migrations without explicit user approval
- Printing connection strings while “verifying” migrate readiness
- Re-pushing babysit fixes without re-running Phase 1.5 when the ship set changed

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/issue` | Files thorough Linear tickets only; Phase 1.5 spawns it per actionable finding |
| `/solve` | Implements Linear issue(s) onto local `dev` and **pushes `origin/dev` after checks** (`/solve [N\|all]`, default 1; no main/prod); Phase 1.5 may spawn it to close the review loop |
| `/review` | Optional local/branch/PR review tooling; **not** the `/prb` ship gate (Phase 1.5 is local-only and does not post GitHub PENDING reviews) |
| `/pr-babysit` | Watches arbitrary PR numbers; does not define the push-`dev`/open-`main` flow |
| `/yeet` | Same ship set, **no** panel / CI wait; still runtime-proof in-scope; merge immediately |
| `/prb` | End-to-end: session → **four-agent grok-4.6 review gate + closed-loop fix** → `origin/dev` → PR into `main` → timed babysit → **project production migrate when needed** → merge |
