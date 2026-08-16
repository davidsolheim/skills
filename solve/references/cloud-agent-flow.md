# Cursor cloud agent flow (`/solve` + cloud land-to-`dev`)

Canonical procedure for **Cursor cloud agents** (Dev Bot launches, remote Cloud Agents).  
**Not** for local Grok CLI worktrees — those may use sequential local delivery or Mac/Grok-only [`fast-mode.md`](fast-mode.md).

Parent skill: [`../SKILL.md`](../SKILL.md)  
Git baseline + hard gate: [`git-dev-workflow.md`](git-dev-workflow.md)  
Claims / Linear: [`multiplayer-linear.md`](multiplayer-linear.md)  
`/prb` Path A (cloud land) + Path B (local ship to main): [`../../prb/SKILL.md`](../../prb/SKILL.md)

---

## Purpose

One cloud agent per Linear leaf:

1. **Hard refresh** `origin` so `origin/dev` contains latest `origin/main`.
2. Create a **Linear-named** issue branch from that tip (never agent names, never `solve/<run>/…`).
3. Implement → verify → push the **same** issue branch.
4. Open **one** PR: **base `dev` ← head issue-branch**. Iterate on that PR until CI is green **and** zero open useful review comments.
5. **Merge that PR into `origin/dev`**. Linear → **In Review**. Comment the merge SHA.
6. **Never** open a PR against `main`. **Never** merge to `main`. **Never** mark Linear **Done** (`/prb` Path B owns `dev` → `main` + Done).

---

## Runtime detection

Use this flow when **any** of:

- Running as a **Cursor cloud agent** / remote Cloud Agent / Dev Bot launch
- Launch prompt or environment says cloud / `starting_ref` / no interactive Mac checkout
- User or orchestrator says “cloud solve” / “Cursor cloud”

Do **not** use Grok fast-mode wave PRs (`origin/solve/<run>/<issue>`) on this path.

---

## Hard pre-branch gate (non-negotiable)

**Before creating or launching onto a new origin issue branch**, complete every step. This is a **hard rule**, not a suggestion. Skipping it is a skill failure.

Works from a Dev Bot with **no local Mac checkout** — use `gh` + remote refs only when needed.

```text
1. git fetch origin
   (or: gh api / equivalent that updates remote-tracking knowledge of origin)

2. Fast-forward / update local knowledge of origin/main and origin/dev:
   - Confirm refs: git rev-parse origin/main origin/dev
   - If origin/dev is missing: create it from origin/main once
     (git push origin origin/main:refs/heads/dev  OR checkout/push locally)

3. If origin/main is NOT an ancestor of origin/dev:
     merge origin/main into dev and push origin/dev
     Verify: git merge-base --is-ancestor origin/main origin/dev

4. If origin/dev moved (step 3 or concurrent push), use the NEW tip.
   Record: DEV_TIP=$(git rev-parse origin/dev)

5. Create the issue branch FROM that tip only.
   Cloud launch starting_ref = origin/dev  (or the SHA of DEV_TIP).
   Never branch from stale main, stale dev, or an old SHA.

6. Already shipped?
   If the issue’s commits (or equivalent fix) are already on origin/dev OR origin/main:
     - Comment the SHA on Linear
     - Skip — do not re-implement
```

### Dev Bot launch checklist (no Mac)

```text
A. gh auth / git remote available in the agent environment
B. git fetch origin   (or fetch via gh + update refs)
C. Ensure origin/dev exists and contains origin/main
     if ! git merge-base --is-ancestor origin/main origin/dev; then
       # merge main → dev on a checkout of dev, then:
       git push origin dev
     fi
D. DEV_TIP=$(git rev-parse origin/dev)
E. Launch Cursor cloud agent with:
     starting_ref: origin/dev
     # or starting_ref: <DEV_TIP sha>
F. Agent creates Linear-named branch from that ref (step below)
```

---

## Branch naming

| Allowed | Forbidden |
| --- | --- |
| Linear identifier as-shown (e.g. `KEC-799`) | Agent / bot / CLI names (`cursor-…`, `grok-…`, `codex-…`) |
| Linear `gitBranchName` when present | `solve/<run>/<issue>` (Grok worktree isolation only) |
| `feat/<prefix-lower>-<number>-short-slug` if neither above | Wave branches `solve/<run>/wave-N` for cloud |

Prefer **`gitBranchName`**, else the issue id (`KEC-799`), else `feat/…`.

---

## Per-issue delivery (one PR)

```text
Phase C0  Hard pre-branch gate (above) — MUST pass
Phase C1  Claim Linear leaf (multiplayer-linear.md CAS)
Phase C2  Create/checkout issue branch from DEV_TIP; push -u origin HEAD
Phase C3  Implement → review → verify on that branch only
Phase C4  Open or reuse ONE PR: --base dev --head <issue-branch>
Phase C5  Babysit: fix CI + useful review comments on the SAME branch/PR
          (zero open useful threads; no waive)
Phase C6  Merge PR into origin/dev (cloud “PRB into origin/dev”)
Phase C7  Linear In Review + comment merge SHA; never Done; never main
```

### C4 — One PR per issue

```bash
# Reuse if already open
gh pr list --state open --head "$ISSUE_BRANCH" --base dev --json number,url

# Else create once
gh pr create --base dev --head "$ISSUE_BRANCH" \
  --title "<ISSUE>: <short title>" \
  --body "$(cat <<'EOF'
## Summary
- Implements <ISSUE>

## Test plan
- [ ] CI green
- [ ] Zero open useful review comments
- [ ] Merge into `dev` (not `main`) when ready

## Notes
- Head: issue branch → Base: `dev`
- Opened by Cursor cloud `/solve`
- Production / `main`: **/prb** (Path B) only — not this agent
EOF
)"
```

**Do not** open a pile of comment-fix PRs. Push fixes to the **same** head branch until the issue is fully resolved.

### C5 — Done-enough to land on `dev`

All must be true before merge:

1. PR still open; base is still **`dev`**; head is still the issue branch
2. `mergeable` / not CONFLICTING (rebase/merge `origin/dev` into the issue branch if `dev` moved; re-verify)
3. Required CI green (or repo has no required checks and none failed)
4. **Zero** open useful review comments/threads (same usefulness bar as `/prb` babysit — no waive)
5. `reviewDecision` is not `CHANGES_REQUESTED`

### C6 — Merge into `origin/dev`

```bash
gh pr merge "$PR_NUMBER" --merge   # or repo default; never into main
git fetch origin
# confirm issue is on origin/dev
git merge-base --is-ancestor <issue-merge-sha> origin/dev
```

No user approval needed for `origin/dev`. **Never** `gh pr merge` into `main`.

### C7 — Linear closeout

1. Completion comment: PR URL, **`origin/dev` merge SHA**, verify evidence.
2. State → **In Review** (or stay In Progress if the team has no In Review).
3. **Never Done** — `/prb` Path B marks Done after merge to `main`.

---

## Multi-issue / drain on cloud

- Prefer **one cloud agent (or one launch) per leaf**.
- Orchestrators that launch many agents: each launch must pass the **hard pre-branch gate** and set `starting_ref` to the **current** `origin/dev` tip (re-fetch between launches if a prior PR just merged).
- Soft deps: do not launch a dependent leaf until its hard dep is **merged to `origin/dev`**.
- Do **not** use Grok `solve/<run>/…` wave PRs on Cursor cloud.

---

## Relation to `/prb`

| Path | Who | What |
| --- | --- | --- |
| **A — Cloud land** | Cursor cloud `/solve` (this file) | Issue branch → one PR → merge **`origin/dev`** → Linear In Review |
| **B — Local ship** | Mac `/prb` | `origin/dev` → PR → **`main`** (+ migrate, explicit approval) → Linear **Done** |

Cloud agents **must not** run Path B (no PR/merge to `main`).

---

## Anti-patterns (cloud)

- Creating an issue branch **before** `git fetch` + ensuring `origin/dev` contains `origin/main`
- Branching from stale `main`, stale `dev`, or an old SHA
- `starting_ref` pointing at `main` or a random prior commit when `origin/dev` exists
- Naming branches after agents or using `solve/<run>/…` on cloud
- Opening PRs with **base `main`**
- Opening multiple fix PRs instead of iterating the same issue PR
- Stopping at “In Review” / green CI **without** merging the issue PR into `origin/dev` once the issue is actually done
- Marking Linear **Done** from cloud
- Force-pushing `main` (or casually force-pushing `dev`)
- Re-implementing an issue already present on `origin/dev` / `origin/main`
