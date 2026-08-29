# /prb — Local Code Review Gate + Closed-Loop Fix Cycle

When `/prb` has finished Phase 1 (local `dev` contains latest `origin/main` and the session ship set), **run this gate before any push to `origin/dev` and before opening a PR**. The gate is an **intensity-selected review panel** on `grok-4.6` (bands: [`../../docs/intensity.md`](../../docs/intensity.md)), merged through [`review-rubric.md`](review-rubric.md). Findings live in scratch markdown. A `prb-fixer` applies them on local `dev`. The gate re-runs until clean (or the cycle cap). **One** Linear comment when the gate finishes.

Authority for the ship-level flow remains [`../SKILL.md`](../SKILL.md). This file is the detailed procedure for **Phase 1.5**. Rubric and specialist prompts are not duplicated here.

---

## 1. Purpose and hard rules

| Rule | Detail |
|------|--------|
| **When** | After Phase 1 (1A–1C). **Before** Phase 1D push and Phase 2 PR. |
| **Pass** | Zero **actionable** findings on `origin/main...dev` **and** runtime proof for in-scope ships ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)). |
| **Fail / block** | Actionable findings remain after `--max-fix-cycles` (default 2), or the fixer / thoroughness agent cannot complete. **Do not push. Do not open a PR.** |
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
| `--max-fix-cycles N` | Cap full **review → fix → re-review** cycles. Default **2**. |

Parse from the `/prb` invocation. Record:

| Field | Default |
|-------|---------|
| `SKIP_REVIEW` | false |
| `SHIP_INTENSITY` | from [`../../docs/intensity.md`](../../docs/intensity.md) (`max` of stamps; bump critical on auth/billing/schema/migrations in the diff) |
| `EXHAUSTIVE_REVIEW` | true only when `SHIP_INTENSITY` is **critical**, unless `--exhaustive-review` / `--no-exhaustive` |
| `MAX_FIX_CYCLES` | 2 |
| `REVIEW_CYCLES_USED` | 0 |
| `FINDINGS_FIXED_COUNT` | 0 |
| `GATE_COMMENT_ISSUE` | first `SHIP_LINEAR_ID` or empty |
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
5. When a finding is a rule violation, **cite** the rule (heading + short quote or bullet paraphrase) in the finding.

If no `## Code Review Rules` sections exist, still review for correctness/security/performance/maintainability/tests; do not invent fake rule citations.

---

## 5. What is actionable

**Authority:** [`review-rubric.md`](review-rubric.md). Do not invent a second policy here.

**Actionable** (blocks push, goes to the fixer): `[P0]` / `[P1]`, plus `[P2]` that is high-signal correctness, security, or regression.

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

- `subagent_type`: `prb-reviewer` (fallback `general-purpose` if the host rejects the type; say so once)
- `model`: `grok-4.6` (required for this gate; never omit; [`../../docs/grok-models.md`](../../docs/grok-models.md))
- `background`: `true`
- `description`: `[<tag>] prb local gate c${C} r${R}`
- Do **not** pass `capability_mode`
- Do **not** pass a fake `effort:` field. Medium reasoning comes from the `prb-reviewer` role.

Then wait with `get_command_or_subagent_output` until the spawned roles complete (or the non-fatal specialists if one of those failed).

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
- Panel: <roles spawned> · grok-4.6 · ship intensity: <band>
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

`--no-exhaustive`: exactly **one** panel pass per cycle. Never run a third panel pass in the same cycle. Closed-loop re-review after a fixer commit is a new cycle, not an exhaustive extra pass.

---

## 7. Closed-loop fix cycle

Scratch is the ticket. Linear is one comment at the end.

### Algorithm

```text
if SKIP_REVIEW:
  REVIEW_EXIT = skipped
  run runtime proof ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)) on in-scope ship set
  if proof fails: DENY_PUSH
  else: post_gate_comment(); return allow_push

cycle = 0
findings_fixed = 0

loop:
  findings = run Local Code Review (section 6) on origin/main...dev
  actionable = filter_actionable(findings)

  if actionable is empty:
    run runtime proof ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)) on in-scope ship set
    if proof fails:
      treat as actionable (re-enter fixer, or DENY_PUSH); do not allow_push
    REVIEW_EXIT = clean
    REVIEW_CYCLES_USED = cycle
    FINDINGS_FIXED_COUNT = findings_fixed
    post_gate_comment()
    return allow_push

  if cycle >= MAX_FIX_CYCLES:
    REVIEW_EXIT = blocked-at-cap
    post_gate_comment()  # leftovers listed
    return DENY_PUSH

  cycle += 1

  # One fixer, whole actionable list
  spawn prb-fixer (model grok-4.6) with merged c${C}.md as the contract
  fixer writes sibling prb-review-${RUN_ID}-c${C}-fixes.md
  orchestrator:
    - confirm product diff is on local dev
    - commit product files only (HEREDOC; never stage $scratch_dir)
    - rm that cycle’s scratch JSON/md (including fixes.md after copying
      a 5–10 line “what changed” into orchestrator memory for the gate comment)
    findings_fixed += count of actionable items the fixes.md claims
  if fixer fails: leave findings open; may retry once this cycle

  ensure checkout is local dev
  if origin/main may have moved: git fetch; re-merge origin/main into dev (Phase 1A–1B)
  goto loop  # re-review
```

### `prb-fixer` prompt (required)

```markdown
You are the /prb closed-loop fixer.

Read the gate artifact in full:
<abs path to prb-review-${RUN_ID}-c${C}.md>

Fix every item under **Actionable findings**. Root cause, not a plaster.
Add or update tests when the defect needs them. Do not nit-hunt.
Do not call Linear. Do not push. Do not stage scratch files.

Work on local **dev** only.

When done, append to <abs path to …-c${C}-fixes.md>:
- Finding id → what you changed (path + one sentence)
- Tests/commands you ran
- Anything you could not fix (and why)
```

If two findings touch the same files, one fixer still does both. Spawn a second fixer only when `primary_paths` are disjoint **and** the first fixer has committed.

### Sequencing

| Step | Parallelism |
|------|-------------|
| Review | Intensity-selected `grok-4.6` `prb-reviewer` agents in parallel, then orchestrator merge |
| Fix | **One** `prb-fixer` (medium via the role) |
| Re-review | Full panel again, only after the cycle’s fixer finished or failed |

### Re-review discipline

- Always re-run the full Local Code Review after a fix cycle — do **not** assume findings are gone.
- Deduplicate: if the same defect reappears, keep the same F-id in the next merged markdown.
- New findings discovered on re-review enter the next cycle (counts toward `MAX_FIX_CYCLES`).

### `post_gate_comment`

Authority: [`linear-ship-comments.md`](linear-ship-comments.md) template C.

- Issue: first `SHIP_LINEAR_ID`. None → skip Linear; put the body in Phase 5.
- `list_comments` first. Skip if `/prb — local review gate` already exists for this `dev` SHA.
- Body: actionable list, what landed, commit SHA(s), leftovers. No secrets.

---

## 8. Mid-ship re-entry (babysit fix-push)

When Phase 3 babysit applies a fix that changes local `dev` and needs re-push:

1. `git fetch origin`; ensure local `main` matches `origin/main`; **merge `origin/main` into local `dev`**.
2. **Re-run Phase 1.5** on the updated `origin/main...dev` (unless `--skip-review` was set for the whole `/prb` run). Prefer **delta**: re-run only specialists whose area changed (a copy fix does not relaunch security + rules + challenge). Recompute `SHIP_INTENSITY` if the ship set grew.
3. Only then `git push origin dev` and restart the quiet timer.

Do not treat the first clean review as a permanent waiver for later fix commits.

---

## 9. Exit criteria and report fields

| `REVIEW_EXIT` | Push allowed? | Report line shape |
|---------------|---------------|-------------------|
| `clean` | **Yes** (if main-ancestor checks also pass) | `Review: local clean after N cycles` and/or `local fixed M findings from scratch` |
| `skipped` | **Yes** (with loud warning) | `Review: skipped (--skip-review)` |
| `blocked-at-cap` | **No** | `Review: blocked at cycle cap (remaining …)` |
| `blocked-tooling` | **No** | `Review: blocked (fixer failure)` or `blocked (thoroughness agent failed)` |

Always include:

```text
Gate comment: /prb — local review gate on TEAM-123 | none (no ship ids)
Findings fixed: M
```

---

## 10. Anti-patterns (Phase 1.5)

- Pushing or opening a PR with actionable findings still open
- Skipping the gate without `--skip-review`
- Filing Linear issues or nested `/solve` for gate findings
- Fixing in the orchestrator by hand instead of `prb-fixer`
- Staging `$TMPDIR` review scratch or committing it
- Parallel fixers racing merges onto `dev` without an orchestrator
- Counting nits as gate failures — or ignoring critical/serious as “later”
- Skipping re-review after fixer commits
- Pushing from an issue branch instead of local `dev`
- Printing secrets into review artifacts or Linear bodies
- Weakening the `origin/main` → `dev` merge requirement “because review passed”
- Orchestrator writing findings instead of spawning the panel
- Spawning a 4-agent panel on a light ship, or skipping security on a critical ship
- Passing a fake `effort:` field on `spawn_subagent`
- Using any model other than `grok-4.6` for these reviewers (including inherit-parent / Claude / GPT / Composer)
- Treating the panel as a nit hunt — or treating P0/P1 as optional
- Running three sequential single-reviewer passes instead of the intensity-selected panel
- Running exhaustive review on a non-critical ship without `--exhaustive-review`
- Posting GitHub PENDING reviews from this gate
- Allowing push because the panel is clean while in-scope runtime proof was skipped or unproven
- Treating `--skip-review` as a waiver of [`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)

---

## 11. Relation to other skills

| Skill | Role here |
|-------|-----------|
| `/issue` | **Not used** in this gate |
| `/solve` | Cheap construction onto local `dev`; **not** this gate |
| `prb-fixer` | Applies the merged scratch markdown on local `dev` |
| `/review` | Optional local/PR review tooling; **not** this gate; do not post GitHub PENDING reviews from Phase 1.5 |
| [`review-rubric.md`](review-rubric.md) | Finding policy (what to flag, priority, JSON schema) |
| [`reviewer-prompts.md`](reviewer-prompts.md) | Specialist overlays prepended after the rubric |
| `/prb` Phase 3 babysit | Separate CI/bot loop **after** clean local review + push + PR |
