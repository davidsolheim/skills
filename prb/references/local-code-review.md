# /prb — Local Code Review Gate + Closed-Loop Fix Cycle

When `/prb` has finished Phase 1 (local `dev` contains latest `origin/main` and the session ship set), **run this gate before any push to `origin/dev` and before opening a PR**. Findings become Linear issues via `/issue`, are fixed on local `dev` via `/solve`, then the gate re-runs until clean (or the cycle cap).

Authority for the ship-level flow remains [`../SKILL.md`](../SKILL.md). This file is the detailed procedure for **Phase 1.5**.

---

## 1. Purpose and hard rules

| Rule | Detail |
|------|--------|
| **When** | After Phase 1 (1A–1C). **Before** Phase 1D push and Phase 2 PR. |
| **Pass** | Zero **actionable** findings on `origin/main...dev`. |
| **Fail / block** | Actionable findings remain after `--max-fix-cycles` (default 5), or `/issue` / `/solve` cannot complete. **Do not push. Do not open a PR.** |
| **Skip** | Only with explicit `--skip-review`. Loud warning in the final report. Never invent skip. |
| **Fixes land on** | **Local `dev` only.** No `git push` inside the closed loop. |
| **Trunk still required** | This gate does **not** replace merge-of-`origin/main`. If `origin/main` moves mid-loop, re-fetch and re-merge into local `dev` before re-review or push. |

---

## 2. Flags (Phase 1.5)

| Flag | Effect |
|------|--------|
| `--skip-review` | Skip entire gate + closed loop. Dangerous. Explicit only. |
| `--exhaustive-review` | Force exhaustive mode **on** (default is already on). |
| `--no-exhaustive` | Single review pass per cycle (still blocks on any actionable findings found). |
| `--max-fix-cycles N` | Cap full **review → issue → solve → re-review** cycles. Default **5**. |

Parse from the `/prb` invocation. Record:

| Field | Default |
|-------|---------|
| `SKIP_REVIEW` | false |
| `EXHAUSTIVE_REVIEW` | true (unless `--no-exhaustive`) |
| `MAX_FIX_CYCLES` | 5 |
| `REVIEW_CYCLES_USED` | 0 |
| `LINEAR_ISSUES_CREATED` | [] |
| `LINEAR_ISSUES_SOLVED` | [] |
| `FINDINGS_FIXED_COUNT` | 0 |
| `REVIEW_EXIT` | unset → later `clean` \| `skipped` \| `blocked-at-cap` \| `blocked-tooling` |

---

## 3. Ship set (what to review)

From the git root, on local `dev` after Phase 1:

```bash
# Commits that would ship into main
git log origin/main..dev --oneline

# Full three-dot diff (merge-base aware) — primary review target
git -c core.quotepath=false diff origin/main...dev

# Paths only
git -c core.quotepath=false diff --name-only origin/main...dev
```

Also useful:

```bash
git diff --stat origin/main...dev
git merge-base --is-ancestor origin/main dev   # must still be true
```

### Empty or trivial ship

- If Phase 0 already determined there is nothing to ship, do not invent a review.
- If the only change is an empty merge with **no** unique commits vs `origin/main`, there is nothing to push; exit earlier.
- Otherwise always review the **actual** `origin/main...dev` diff that would enter the PR.

### Size gate

If the unified diff is huge (> ~1 MB text), still review, but prefer path-by-path / subagent review of high-risk areas first (auth, payments, migrations, data access, public API). Do not skip the gate because the diff is large unless the user passed `--skip-review`.

---

## 4. Load AGENTS.md Code Review Rules

Before scoring findings:

1. Read **root** `AGENTS.md` / `Agents.md` / `CLAUDE.md` if present.
2. For every path in `git diff --name-only origin/main...dev`, walk parent directories and load nested `AGENTS.md` that apply to those trees (same pattern as package-scoped agent rules).
3. Extract every section titled (case-insensitive match OK):
   - `## Code Review Rules`
   - `## Code review rules`
   - nearby equivalents clearly meant as review invariants (`## Review checklist` only if it is the repo’s stated review bar)
4. Treat those bullets as **must-apply** invariants for this gate.
5. When a finding is a rule violation, **cite** the rule (heading + short quote or bullet paraphrase) in the finding and in the Linear issue body.

If no `## Code Review Rules` sections exist, still review for correctness/security/performance/maintainability/tests; do not invent fake rule citations.

---

## 5. Review focus (high-signal only)

**Prioritize (actionable when real):**

| Category | Examples |
|----------|----------|
| Correctness / regressions | Wrong conditionals, broken control flow, off-by-one, incomplete migrations of call sites, race conditions |
| Security | Authz bypass, secret leakage, injection, unsafe deserialization, SSRF, missing CSRF on state-changing routes where required |
| Performance | N+1 queries, unbounded loads, accidental full-table scans introduced by the diff, blocking work on hot paths |
| Maintainability (high impact) | Clear AGENTS violations, logic in the wrong layer, destructive coupling introduced by the ship |
| Tests | New behavior with no tests when the repo expects them; deleted tests without replacement |
| Repo invariants | Doppler/env rules, no `db:push` to prod patterns, branch/package boundaries from AGENTS |

**Deprioritize / do not block the gate:**

- Pure style / formatting / import order
- Optional renames, “could be slightly cleaner” without correctness risk
- Drive-by refactors outside the ship set
- Nits that would not matter if CI is green and behavior is correct

**Severity labels** (use these in structured output):

| Severity | Meaning | Actionable for gate? |
|----------|---------|----------------------|
| `critical` | Breaks production / security / data integrity | **Yes** |
| `serious` | Likely bug, regression, or hard AGENTS violation | **Yes** |
| `medium` | Real risk but narrower; correctness/security still counts | **Yes** if high-signal correctness/security/regression; else optional |
| `nit` | Style / low-value | **No** — log only; do not open Linear issues |

**Gate pass = zero actionable findings.** Nits alone never block push.

---

## 6. How to run the review

### Prefer a read-only reviewer subagent

Spawn `general-purpose` (or equivalent) with description prefix `[reviewer] prb local gate`:

- **Read-only:** no edits, no git writes, no Linear, no push.
- Prompt must include: ship-set commands, path list, AGENTS Code Review Rules text, exhaustive flag, severity policy, output path.
- Instruct: read the diff **and** surrounding source for call sites/types before flagging.

Orchestrator may perform the review itself when the diff is tiny; prefer subagent for non-trivial ships.

**Do not** use the bundled `/review` PR-posting flow (no GitHub PENDING review). This gate is local-only.

### Exhaustive mode (default **on**)

When `EXHAUSTIVE_REVIEW=true`:

```text
round = 0
MAX_EXHAUSTIVE_ROUNDS = 3
findings = []

while round < MAX_EXHAUSTIVE_ROUNDS:
  round += 1
  new = review_pass(ship_set, known=findings)
  actionable_new = filter_actionable(new)
  if actionable_new is empty:
    break
  merge into findings (dedupe by file+line+problem signature)
  # next round: hunt for issues NOT already listed
```

When `--no-exhaustive`: exactly **one** review pass per cycle.

Still: one thorough pass beats a flood of nits. Stop adding findings when only low-value items remain.

### Structured output (required)

Write a review artifact (mode 0600) under a user-private scratch dir:

```bash
umask 077
scratch_dir="${TMPDIR:-/tmp}/grok-$(id -u)"
mkdir -p "$scratch_dir" && chmod 700 "$scratch_dir"
# e.g. ${scratch_dir}/prb-review-${RUN_ID}-c${cycle}.md
```

**Template:**

```markdown
# Local code review — origin/main...dev
- Repo: <owner/repo>
- dev SHA: <sha>
- origin/main SHA: <sha>
- Exhaustive: yes|no · round: r/R
- Correctness verdict: pass | fail
- Confidence: high | medium | low

## Summary
<2–4 sentences>

## Actionable findings

### F1 — Severity: critical|serious|medium
- File: path/to/file.ext:START-END
- Explanation: <what is wrong and why it matters>
- AGENTS rule: <cite or "none">
- Suggested fix shape: <one short approach, not a full patch>

## Non-blocking notes (nits)
- …

## Verdict
- Actionable count: N
- Gate: CLEAN | NEEDS_FIX
```

Never put secrets, tokens, connection strings, or full `.env` contents in the artifact.

---

## 7. Closed-loop fix cycle

### Algorithm

```text
if SKIP_REVIEW:
  REVIEW_EXIT = skipped
  return allow_push

cycle = 0
all_created = []
all_solved = []
findings_fixed = 0

loop:
  findings = run Local Code Review (section 6) on origin/main...dev
  actionable = filter_actionable(findings)

  if actionable is empty:
    REVIEW_EXIT = clean
    REVIEW_CYCLES_USED = cycle
    LINEAR_ISSUES_* = all_created / all_solved
    FINDINGS_FIXED_COUNT = findings_fixed
    return allow_push

  if cycle >= MAX_FIX_CYCLES:
    REVIEW_EXIT = blocked-at-cap
    report outstanding actionable + Linear ids so far
    return DENY_PUSH

  cycle += 1

  # One Linear issue per distinct problem (group only tightly related)
  for each distinct finding (or tight group) in actionable:
    1. Spawn subagent: run /issue skill with a precise description
       (file/line, severity, AGENTS rule, explanation, suggested fix shape,
        note: "Filed by /prb Phase 1.5 local review gate before push")
    2. Capture TEAM-### (+ URL); append to all_created
    3. If issue create fails → REVIEW_EXIT = blocked-tooling; DENY_PUSH
    4. Spawn subagent: run /solve TEAM-### (local dev only; no push/PR)
       Prefer sequential solve so merges to dev do not race.
    5. On solve success: append to all_solved; findings_fixed += 1
       On solve failure: leave finding open; do not mark fixed;
       may retry once; if still failing, continue other findings this cycle
       but DENY_PUSH if any actionable remain after re-review

  # After this cycle's solves:
  ensure checkout is local dev
  if origin/main may have moved: git fetch; re-merge origin/main into dev (Phase 1A–1B)
  goto loop  # re-review
```

### `/issue` subagent expectations

- Follow the `/issue` skill end-to-end (team/project resolve, code map, acceptance criteria).
- Title should be problem-focused and ship-blocking clear.
- Priority: map `critical` → Urgent/High, `serious` → High, `medium` → Medium.
- Body must include file/line evidence and AGENTS rule citation when applicable.
- **One problem per issue** unless findings are the same root cause in the same function.

### `/solve` subagent expectations

- Follow `/solve TEAM-123` (or equivalent single-id invocation).
- Delivery = **merge fix into local `dev` only** — no push, no PR (solve default).
- After solve: confirm the fix commit(s) are on local `dev`.
- Do not start Phase 1D push from inside solve.

### Sequencing

| Step | Parallelism |
|------|-------------|
| Review | One reviewer at a time per cycle |
| `/issue` creates | Parallel OK for independent findings |
| `/solve` | **Default sequential** (one after another on local `dev`) |
| Re-review | Only after the cycle’s intended solves finished or failed |

### Re-review discipline

- Always re-run the full Local Code Review after a fix cycle — do **not** assume findings are gone.
- Deduplicate: if the same issue reappears, keep the existing Linear id when possible; file a follow-up only if the first fix was incomplete and needs a new ticket.
- New findings discovered on re-review enter the next cycle (counts toward `MAX_FIX_CYCLES`).

---

## 8. Mid-ship re-entry (babysit fix-push)

When Phase 3 babysit applies a fix that changes local `dev` and needs re-push:

1. `git fetch origin`; ensure local `main` matches `origin/main`; **merge `origin/main` into local `dev`**.
2. **Re-run this entire Phase 1.5 gate** on the updated `origin/main...dev` (unless `--skip-review` was set for the whole `/prb` run).
3. Only then `git push origin dev` and restart the quiet timer.

Do not treat the first clean review as a permanent waiver for later fix commits.

---

## 9. Exit criteria and report fields

| `REVIEW_EXIT` | Push allowed? | Report line shape |
|---------------|---------------|-------------------|
| `clean` | **Yes** (if main-ancestor checks also pass) | `Review: local clean after N cycles` and/or `local fixed M findings across K Linear issues` |
| `skipped` | **Yes** (with loud warning) | `Review: skipped (--skip-review)` |
| `blocked-at-cap` | **No** | `Review: blocked at cycle cap (remaining …)` |
| `blocked-tooling` | **No** | `Review: blocked (Linear/solve failure)` |

Always include when issues were filed:

```text
Linear issues created & solved: TEAM-123, TEAM-124, … | none
```

Distinguish created vs solved if they differ (e.g. created 3, solved 2 → list both clearly and **deny push**).

---

## 10. Anti-patterns (Phase 1.5)

- Pushing or opening a PR with actionable findings still open
- Skipping the gate without `--skip-review`
- One mega Linear issue for unrelated findings
- Fixing in the orchestrator by hand instead of `/issue` + `/solve` when Linear is available (exception: Linear outage → stop and report drafts; still do not push dirty review)
- Parallel `/solve` workers racing merges onto `dev` without an orchestrator
- Counting nits as gate failures — or ignoring critical/serious as “later”
- Skipping re-review after solves
- Pushing from an issue branch instead of local `dev`
- Printing secrets into review artifacts or Linear bodies
- Weakening the `origin/main` → `dev` merge requirement “because review passed”

---

## 11. Relation to other skills

| Skill | Role here |
|-------|-----------|
| `/issue` | File one implementation-ready Linear ticket per distinct finding |
| `/solve` | Implement ticket onto **local `dev`** (no push) |
| `/review` | Optional inspiration for reviewer structure; **not** required; do not post GitHub PENDING reviews from this gate |
| `/prb` Phase 3 babysit | Separate CI/bot loop **after** clean local review + push + PR |
