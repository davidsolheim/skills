# /prb — Local Code Review Gate + Closed-Loop Fix Cycle

When `/prb` has finished Phase 1 (local `dev` contains latest `origin/main` and the session ship set), **run this gate before any push to `origin/dev` and before opening a PR**. The gate is an **intensity-selected review panel** on `grok-4.6` (bands: [`../../docs/intensity.md`](../../docs/intensity.md)), merged through [`review-rubric.md`](review-rubric.md). Findings become Linear issues via `/issue`, are fixed on local `dev` via `/solve` at the **finding’s** effort, then the gate re-runs until clean (or the cycle cap).

Authority for the ship-level flow remains [`../SKILL.md`](../SKILL.md). This file is the detailed procedure for **Phase 1.5**. Rubric and specialist prompts are not duplicated here.

---

## 1. Purpose and hard rules

| Rule | Detail |
|------|--------|
| **When** | After Phase 1 (1A–1C). **Before** Phase 1D push and Phase 2 PR. |
| **Pass** | Zero **actionable** findings on `origin/main...dev` **and** runtime proof for in-scope ships ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)). |
| **Fail / block** | Actionable findings remain after `--max-fix-cycles` (default 2), or `/issue` / `/solve` cannot complete. **Do not push. Do not open a PR.** |
| **Skip** | Only with explicit `--skip-review`. Loud warning in the final report. Never invent skip. |
| **Fixes land on** | **Local `dev` only.** No `git push` inside the closed loop. |
| **Trunk still required** | This gate does **not** replace merge-of-`origin/main`. If `origin/main` moves mid-loop, re-fetch and re-merge into local `dev` before re-review or push. |

---

## 2. Flags (Phase 1.5)

| Flag | Effect |
|------|--------|
| `--skip-review` | Skip entire gate + closed loop. Dangerous. Explicit only. |
| `--exhaustive-review` | After a clean first panel, run **one** more panel pass hunting for issues not already listed (default **on only for critical ships**). |
| `--no-exhaustive` | Exactly one panel pass per cycle (still blocks on any actionable findings found). |
| `--max-fix-cycles N` | Cap full **review → issue → solve → re-review** cycles. Default **2**. |

Parse from the `/prb` invocation. Record:

| Field | Default |
|-------|---------|
| `SKIP_REVIEW` | false |
| `SHIP_INTENSITY` | from [`../../docs/intensity.md`](../../docs/intensity.md) (`max` of stamps; bump critical on auth/billing/schema/migrations in the diff) |
| `EXHAUSTIVE_REVIEW` | true only when `SHIP_INTENSITY` is **critical**, unless `--exhaustive-review` / `--no-exhaustive` |
| `MAX_FIX_CYCLES` | 2 |
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

If the unified diff is huge (> ~1 MB text), still run the intensity-selected panel. Tell every agent to cover high-risk paths first (auth, payments, migrations, data access, public API) and then the rest of the ship set. Do not skip the gate because the diff is large unless the user passed `--skip-review`. A huge diff that touches those paths is **critical**.

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

## 5. What is actionable

**Authority:** [`review-rubric.md`](review-rubric.md). Do not invent a second policy here.

**Actionable** (blocks push, files Linear): `[P0]` / `[P1]`, plus `[P2]` that is high-signal correctness, security, or regression.

**Not actionable:** `[P3]` / nits, style, optional cleanup, speculative breakage, pre-existing issues on `origin/main`.

**Gate pass = zero actionable findings.** Nits alone never block push.

---

## 6. How to run the review

Always run the **panel** selected by `SHIP_INTENSITY`. Tiny diffs still get a panel (light = thoroughness only). Orchestrator does **not** author findings.

**Do not** use the bundled `/review` PR-posting flow (no GitHub PENDING review). This gate is local-only.

### 6A. Scratch paths

```bash
umask 077
scratch_dir="${TMPDIR:-/tmp}/grok-$(id -u)"
mkdir -p "$scratch_dir" && chmod 700 "$scratch_dir"
```

Per cycle `C` and panel pass `R` (1 or 2):

| File | Writer |
|------|--------|
| `${scratch_dir}/prb-review-${RUN_ID}-c${C}-r${R}-thoroughness.json` | thoroughness agent |
| `${scratch_dir}/prb-review-${RUN_ID}-c${C}-r${R}-security.json` | security agent |
| `${scratch_dir}/prb-review-${RUN_ID}-c${C}-r${R}-rules.json` | rules agent |
| `${scratch_dir}/prb-review-${RUN_ID}-c${C}-r${R}-challenge.json` | challenge agent |
| `${scratch_dir}/prb-review-${RUN_ID}-c${C}.md` | orchestrator merge (gate artifact) |

Inline these absolute paths into prompts. Never put secrets, tokens, connection strings, or `.env` contents in artifacts.

### 6B. Load the rubric and AGENTS rules

1. Read [`review-rubric.md`](review-rubric.md) in full. Prepend that text to **every** specialist prompt.
2. Load AGENTS Code Review Rules as in §4. Paste the excerpts into every prompt (rules agent still owns citation; others need them as override context).
3. Read [`reviewer-prompts.md`](reviewer-prompts.md) and append **one** specialist overlay per spawn.

### 6C. Spawn the panel (one turn, parallel)

Launch the intensity-selected roles in the same assistant response with `spawn_subagent`:

| `SHIP_INTENSITY` | Roles |
|------------------|--------|
| **light** | thoroughness only |
| **standard** / **heavy** | thoroughness, security, rules, challenge |
| **critical** | thoroughness, security, rules, challenge |

| Tag | Overlay | Fatal if spawn/run fails? |
|-----|---------|---------------------------|
| `[thoroughness]` | Thoroughness | **Yes** — stop the gate; do not push |
| `[security]` | Security | No — warn, continue with remaining |
| `[rules]` | Rules | No — warn, continue with remaining |
| `[challenge]` | Challenge | No — warn, continue with remaining |

Do not spawn roles the band does not call for. Do not skip security on a critical ship.

Shared spawn args:

- `subagent_type`: `general-purpose`
- `model`: `grok-4.6` (required for this gate; never omit; [`../../docs/grok-models.md`](../../docs/grok-models.md))
- `background`: `true`
- `description`: `[<tag>] prb local gate c${C} r${R}`
- Do **not** pass `capability_mode`

Then wait with `get_command_or_subagent_output` until all four complete (or the three non-fatal ones if one specialist failed).

Each agent writes **only** rubric JSON to its output file. If a file is missing or not valid JSON, treat that agent as failed (fatal for thoroughness → `REVIEW_EXIT=blocked-tooling`, do not push; skip for specialists).

### 6D. Merge

Read the JSON files. Build one finding list:

1. Tag each finding with its source (`thoroughness` / `security` / `rules` / `challenge`).
2. Deduplicate by file + overlapping line range + same defect/remedy. Union source tags when merging. Keep the higher priority. When in doubt that they are the same defect, keep both.
3. Drop anything that fails the rubric tests (pre-existing, speculative, intentional, nit-only, author would not fix).
4. Re-check every survivor against applicable `AGENTS.md` rules. Add a citation when the finding is rule-supported; do not drop ordinary bugs that are not rule violations.
5. Classify actionable vs nit using §5.

Write the merged gate artifact:

```markdown
# Local code review — origin/main...dev
- Repo: <owner/repo>
- dev SHA: <sha>
- origin/main SHA: <sha>
- Panel: thoroughness, security, rules, challenge · grok-4.6
- Exhaustive: yes|no · panel pass: r/R
- Agents ok: <list> · failed: <list or none>
- Correctness verdict: pass | fail
- Confidence: high | medium | low

## Summary
<2–4 sentences>

## Actionable findings

### F1 — Severity: critical|serious|medium · P0|P1|P2 · [thoroughness, security]
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

`overall_correctness` of `"patch is incorrect"` from any surviving agent is a fail unless merge dropped every supporting finding.

### 6E. Exhaustive panel (default **on**)

```text
pass = 1
findings = merge(panel_pass(1))
# EXHAUSTIVE_REVIEW default: true only when SHIP_INTENSITY is critical
if EXHAUSTIVE_REVIEW and filter_actionable(findings) is empty:
  pass = 2
  more = merge(panel_pass(2, known=findings))  # prompts include known findings; hunt for NEW issues
  findings = dedupe(findings + more)
```

`--no-exhaustive`: exactly **one** panel pass per cycle. Never run a third panel pass in the same cycle. Closed-loop re-review after `/solve` is a new cycle, not an exhaustive extra pass.

---

## 7. Closed-loop fix cycle

### Algorithm

```text
if SKIP_REVIEW:
  REVIEW_EXIT = skipped
  run runtime proof ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)) on in-scope ship set
  if proof fails: DENY_PUSH
  else: return allow_push

cycle = 0
all_created = []
all_solved = []
findings_fixed = 0

loop:
  findings = run Local Code Review (section 6) on origin/main...dev
  actionable = filter_actionable(findings)

  if actionable is empty:
    run runtime proof ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)) on in-scope ship set
    if proof fails:
      treat as actionable (file /issue + /solve or DENY_PUSH); do not allow_push
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
    1. Spawn subagent (`model: grok-4.6`): run /issue skill with a precise description
       (file/line, severity, AGENTS rule, explanation, suggested fix shape,
        note: "Filed by /prb Phase 1.5 local review gate before push")
    2. Capture TEAM-### (+ URL); append to all_created
    3. If issue create fails → REVIEW_EXIT = blocked-tooling; DENY_PUSH
    4. Spawn subagent (`model: grok-4.6`): run /solve TEAM-### (local dev only; no push/PR)
       Pass `--effort` from the **finding’s** band (default **2** / standard; **5** if the finding is critical). Do not inherit ship-critical for a localized UI/copy fix.
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
| Review | Intensity-selected `grok-4.6` agents in parallel, then orchestrator merge |
| `/issue` creates | Parallel OK for independent findings |
| `/solve` | **Default sequential** (one after another on local `dev`) |
| Re-review | Full panel again, only after the cycle’s intended solves finished or failed |

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
| `blocked-tooling` | **No** | `Review: blocked (Linear/solve failure)` or `blocked (thoroughness agent failed)` |

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
- Orchestrator writing findings instead of spawning the panel
- Spawning a 4-agent panel on a light ship, or skipping security on a critical ship
- Defaulting closed-loop `/solve` to effort 5 for a localized non-critical finding
- Using any model other than `grok-4.6` for these reviewers (including inherit-parent / Claude / GPT / Composer)
- Treating the panel as a nit hunt — or treating P0/P1 as optional
- Running three sequential single-reviewer passes instead of the panel
- Posting GitHub PENDING reviews from this gate
- Allowing push because the panel is clean while in-scope runtime proof was skipped or unproven
- Treating `--skip-review` as a waiver of [`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)

---

## 11. Relation to other skills

| Skill | Role here |
|-------|-----------|
| `/issue` | File one implementation-ready Linear ticket per distinct finding |
| `/solve` | Implement ticket onto **local `dev`** (no push) |
| `/review` | Optional local/PR review tooling; **not** this gate; do not post GitHub PENDING reviews from Phase 1.5 |
| [`review-rubric.md`](review-rubric.md) | Finding policy (what to flag, priority, JSON schema) |
| [`reviewer-prompts.md`](reviewer-prompts.md) | Specialist overlays prepended after the rubric |
| `/prb` Phase 3 babysit | Separate CI/bot loop **after** clean local review + push + PR |
