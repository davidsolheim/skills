# Multiplayer Linear (`/solve` + `/prb`)

Canonical procedure for claiming and closing Linear issues when several Grok CLIs / Cursor cloud agents / Dev Bots share David token. RFC: [`../../docs/rfc-multiplayer-linear.md`](../../docs/rfc-multiplayer-linear.md).

## Unclaimed vs claimed

**Unclaimed:** Backlog/Todo/Triage (or equivalent), not In Progress, no live `claimed-by:` comment newer than 60 minutes.

**Claimed by us:** In Progress (or In Review after we shipped to `origin/dev`) whose latest `claimed-by:` / `stolen-by:` matches this session `run` id.

**Claimed by other:** In Progress with a foreign live claim comment — skip. Do not implement.

## Claim (CAS) — before any git work

1. Eligible leaf (existing 2B–2E) **and unclaimed**.
2. Assign to me if unassigned. Set **In Progress**.
3. Comment, first line exactly:

```
claimed-by: <bot-or-cli> · session <id> · worktree <cwd> · run <RUN_ID|sequential>
```

Include a one-paragraph plan below that line as today. Cloud agents should note delivery = issue PR into `dev` (not main).

4. Re-fetch the issue. If a **different** run `claimed-by:` is newer: abort, do not code, pick another leaf.
5. Fast workers of **one** Mac/Grok orchestrator share `run`. They may claim distinct leaves under that run. Cursor cloud: one claim per leaf/launch.

## Closeout (`/solve` Phase 8)

After the leaf is verified **on `origin/dev`**:

- **Cursor cloud:** issue PR merged into `origin/dev` (CI green + zero open useful review comments). Do not stop at In Review without that merge when the issue is done.
- **Local sequential:** merge + push `origin/dev`.
- **Mac/Grok fast:** wave PR merged into `origin/dev`.

Then:

1. Completion comment with **`origin/dev` SHA** (and PR URL when cloud/fast).
2. Set **In Review** if the team has that status; else stay **In Progress** with `shipped origin/dev @ sha` in the comment.
3. **Never mark Done** here.
4. Epic rollup to Done only when all children are terminal.

On failure: leave In Progress or Blocked; comment; do not Done.

## `/solve all` guard

Before draining: list In Progress on the project. If foreign live claims exist, do **not** start `/solve all` / `/solve all fast`. Use `/solve N` / explicit ids on unclaimed leaves only. One drain orchestrator per project (Mac). Cloud: do not launch a second drain while foreign live claims exist.

## `/prb`

- **Path A (cloud land):** issue branch → `origin/dev` is owned by cloud `/solve` (or Path A procedure). Linear stays **In Review** after that merge — **not Done**.
- **Path B (Mac ship):** Phase 2 PR comment only. Optional In Review. **Not Done** at PR open.
- After merge to `main` (explicit user approval, Path B): **Done** + production ship comment.
- Ignore / do not close foreign In Progress ids.
- Cloud agents must never mark Done and must never merge to `main`.

## `/issues`

File unassigned Backlog/Todo. Never assign as part of filing.
