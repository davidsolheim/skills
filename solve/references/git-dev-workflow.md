# Git workflow for /solve

Long-lived integration branch: **`dev`** only (local + `origin`, all-lowercase **d-e-v** — never capital-`D` `Dev`).  
Trunk: **`main`**. Upstream refresh: **`origin/main`** must be in **`origin/dev`** before any new issue branch.

## Goals

1. Never start issue work on a stale base (**hard pre-branch gate**).
2. Accumulate finished work on **`origin/dev`** (cloud via issue PR; local Mac via merge + push).
3. Never merge to **`main`** from `/solve` — that is `/prb` Path B + explicit user approval.

## Which path?

| Runtime | Procedure |
| --- | --- |
| **Cursor cloud agent** / Dev Bot launch | [`cloud-agent-flow.md`](cloud-agent-flow.md) — hard gate → Linear-named branch → one PR into `dev` → merge → In Review |
| **Local Mac / Grok CLI** (sequential) | Sections 1–7 below — refresh → issue branch → merge local `dev` → push `origin/dev` |
| **Local Mac / Grok `fast` only** | [`fast-mode.md`](fast-mode.md) — **not** the default for Cursor cloud |

---

## Hard pre-branch gate (all runtimes)

**Before creating or launching onto a new origin issue branch**, complete every step. Hard rule — not a suggestion.

```text
1. git fetch origin
2. Update knowledge of origin/main and origin/dev (create origin/dev from origin/main if missing)
3. If origin/main is not an ancestor of origin/dev:
     merge origin/main into dev and push origin/dev
4. Use the current origin/dev tip (DEV_TIP)
5. Create / launch the issue branch from that tip only
   (cloud: starting_ref = origin/dev or DEV_TIP SHA)
6. If the issue is already on origin/dev or origin/main: comment the SHA and skip
```

Verify before branching:

```bash
git fetch origin
git merge-base --is-ancestor origin/main origin/dev   # exit 0 required
DEV_TIP=$(git rev-parse origin/dev)
```

Full cloud / Dev Bot checklist: [`cloud-agent-flow.md`](cloud-agent-flow.md).

---

## Local Mac / sequential procedure

### 1. Snapshot safety

```bash
git status
git branch -vv
```

If unrelated dirty files exist, leave them alone. Do not stash/drop unless necessary and safe; prefer committing only your paths later.

### 2. Refresh main

```bash
git fetch origin
git checkout main
git merge --ff-only origin/main
```

If ff-only fails, report divergence; do not rewrite remote history. Prefer resolving with the user before continuing.

### 3. Update dev from main (and keep origin/dev current)

```bash
# Canonical branch is always lowercase `dev`.
if git show-ref --verify --quiet refs/heads/dev; then
  git checkout dev
elif git show-ref --verify --quiet refs/heads/Dev; then
  # One-time adoption of a legacy misnamed branch — then only use `dev`.
  git branch -m Dev dev
  git checkout dev
else
  git checkout -b dev main
fi
git merge origin/main -m "Merge origin/main into dev"
# If origin/dev was behind main, push so remotes match before issue branches:
git push -u origin dev
```

- First run in a clone: create **`dev`** from current `main` (never create `Dev`).
- Always merge **main → dev**, not the reverse, under this skill.
- Resolve conflicts before starting the Linear issue.
- **Never** use capital-`D` `Dev` for new work, checkouts for issue branches, or merges. If both `dev` and `Dev` exist, use **`dev` only** and note the duplicate to the user (do not delete `Dev` without asking).

### 4. Issue branch

Prefer Linear’s suggested `gitBranchName` when present; else the issue id (e.g. `KEC-799`); else:

```text
feat/<prefix-lower>-<number>-short-slug
```

`<prefix-lower>` is the real Linear team key lowercased (e.g. `eng`, `ops`, `team`). Example: `feat/team-341-profile-avatar`

```bash
git checkout -b feat/team-341-profile-avatar dev
# branch must contain the refreshed origin/dev tip
```

**Never** name branches after agents. **Never** use `solve/<run>/…` on Cursor cloud (Grok fast-mode only).

### 5. Commit on issue branch

After the implement→review loop and verification:

```bash
git add <only-relevant-paths>
git commit -m "$(cat <<'EOF'
TEAM-341: short imperative summary

EOF
)"
```

Include the issue id in the subject line.

### 6. Merge into local `dev` and push `origin/dev`

```bash
git checkout dev
git merge feat/team-341-profile-avatar -m "Merge branch 'feat/team-341-profile-avatar' into dev"
# build + security/verify (repo AGENTS) — do not push until green
git push -u origin dev
```

Confirm `git log --oneline -5` shows the work on `dev` / `origin/dev`.

### 7. Explicit non-goals (local sequential)

```bash
# Do NOT run from /solve:
# gh pr create --base main ...
# gh pr merge ... into main
# Linear Done  (/prb Path B after main)
```

**Cursor cloud** instead opens one PR **into `dev`** and merges that — see [`cloud-agent-flow.md`](cloud-agent-flow.md). Do not push straight to `origin/dev` from cloud without the issue-PR babysit path unless the launch instructions explicitly say otherwise.

## Failure modes

| Situation | Response |
|-----------|----------|
| Dirty conflict with dev←main | Resolve or stop; don’t implement on broken base |
| Issue branch diverged | Rebase onto refreshed `origin/dev` or merge it in, then verify again |
| Accidental commit on main | Move commit onto issue branch / `dev` carefully without destroying user work |
| `origin/main` not in `origin/dev` | **Stop branching** — complete the hard pre-branch gate first |
| Already on `origin/dev` / `main` | Comment SHA; skip re-implement |

## End state of a successful /solve

### Local Mac sequential

- Local **`dev`** / **`origin/dev`** includes latest **`main`** plus this issue’s commits
- Linear **In Review** with `origin/dev` SHA
- **`main`** untouched

### Cursor cloud

- Issue branch created from refreshed **`origin/dev`** tip
- One PR **base `dev`** merged after CI green + zero useful open review comments
- Linear **In Review** with merge SHA
- **`main`** untouched

## Fast mode (`/solve … fast`) — Mac/Grok only

When `FAST_MODE` is true **on local Grok CLI**, follow [`fast-mode.md`](fast-mode.md).

**Cursor cloud agents must not use fast-mode wave PRs** (`origin/solve/<run>/<issue>`). Use [`cloud-agent-flow.md`](cloud-agent-flow.md) instead.
