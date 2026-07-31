# Git workflow for /solve

Long-lived integration branch: **`dev`** only (local, all-lowercase **d-e-v** — never capital-`D` `Dev`).  
Upstream truth for refresh: **`origin/main`** → local **`main`**.

## Goals

1. Never start issue work on a stale base.
2. Accumulate finished work on local **`dev`**.
3. Do **not** push or open PRs unless the user explicitly asks (out of skill default scope).

## Procedure

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

### 3. Update dev from main

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
git merge main -m "Merge main into dev"
```

- First run in a clone: create **`dev`** from current `main` (never create `Dev`).
- Always merge **main → dev**, not the reverse, under this skill.
- Resolve conflicts before starting the Linear issue.
- **Never** use capital-`D` `Dev` for new work, checkouts for issue branches, or merges. If both `dev` and `Dev` exist, use **`dev` only** and note the duplicate to the user (do not delete `Dev` without asking).

### 4. Issue branch

Prefer Linear’s suggested `gitBranchName` when present; else:

```text
feat/<prefix-lower>-<number>-short-slug
```

`<prefix-lower>` is the real Linear team key lowercased (e.g. `eng`, `ops`, `team`). Example only: `feat/team-341-profile-avatar`

```bash
git checkout -b feat/team-341-profile-avatar dev
```

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

### 6. Merge into local dev

```bash
git checkout dev
git merge feat/team-341-profile-avatar -m "Merge branch 'feat/team-341-profile-avatar' into dev"
```

Confirm `git log --oneline -5` shows the merge/work on `dev`.

### 7. Explicit non-goals

```bash
# Do NOT run unless user asked:
# git push origin dev
# git push -u origin feat/...
# gh pr create ...
```

## Failure modes

| Situation | Response |
|-----------|----------|
| Dirty conflict with dev←main | Resolve or stop; don’t implement on broken base |
| Issue branch diverged | Rebase onto dev or merge dev in, then verify again |
| Accidental commit on main | Move commit onto issue branch / dev carefully without destroying user work |
| User asked to push | Confirm once, then push `dev` only if they want that ship path |

## End state of a successful /solve

- Local **`dev`** includes latest **`main`** plus this issue’s commits
- Issue branch merged (or ff’d) into **`dev`**
- No requirement that `origin` knows about `dev` yet

## Fast mode (`/solve … fast`) — parallel addendum

When `FAST_MODE` is true, follow [`fast-mode.md`](fast-mode.md). Git differences:

1. **Orchestrator** refreshes `main` → `dev` once (or per wave) in the **main workspace**.
2. Each worker runs in an isolated **git worktree** (`spawn_subagent` `isolation: worktree`) on a short-lived issue branch based at the **wave base** (`dev` tip when the wave started).
3. Workers **commit only on the issue branch**. They never checkout/merge `dev` in a way that races the orchestrator.
4. Orchestrator integrates with the Subagent Worktree Protocol:
   - `git fetch <worktree_path> HEAD --no-tags`
   - record `commit_sha`; `git cat-file -t`
   - on main workspace: `git checkout dev` then merge issue branch (or cherry-pick `base_sha..commit_sha`) in **merge_order**
   - re-verify on `dev`
5. **Mandatory cleanup after each successful merge (and on failure):**
   ```bash
   # free worktree before deleting branch ref
   grok worktree rm --force <worktree_path>
   git branch -d <issue-branch>   # or -D if already on dev
   # never delete dev or main
   ```
6. Steady-state worktree count ≤ concurrency; **end of run = 0** leftover solve-fast worktrees for the run.
7. Still **no push** unless the user explicitly asked.
