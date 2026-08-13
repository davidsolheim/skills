# Multiplayer Linear (`/solve` + `/prb`)

Canonical procedure for claiming and closing Linear issues when several Grok CLIs / Dev Bots share David token. RFC: [`../../docs/rfc-multiplayer-linear.md`](../../docs/rfc-multiplayer-linear.md).

## Unclaimed vs claimed

**Unclaimed:** Backlog/Todo/Triage (or equivalent), not In Progress, no live `claimed-by:` comment newer than 60 minutes.

**Claimed by us:** In Progress (or In Review after we shipped `dev`) whose latest `claimed-by:` / `stolen-by:` matches this session `run` id.

**Claimed by other:** In Progress with a foreign live claim comment — skip. Do not implement.

## Claim (CAS) — before any git work

1. Eligible leaf (existing 2B–2E) **and unclaimed**.
2. Assign to me if unassigned. Set **In Progress**.
3. Comment, first line exactly:

```
claimed-by: <bot-or-cli> · session <id> · worktree <cwd> · run <RUN_ID|sequential>
```

Include a one-paragraph plan below that line as today.

4. Re-fetch the issue. If a **different** run `claimed-by:` is newer: abort, do not code, pick another leaf.
5. Fast workers of **one** orchestrator share `run`. They may claim distinct leaves under that run.

## Closeout (`/solve` Phase 8)

After verified merge **and** push to `origin/dev`:

1. Completion comment with `origin/dev` SHA (existing evidence list).
2. Set **In Review** if the team has that status; else stay **In Progress** with `shipped origin/dev @ sha` in the comment.
3. **Never mark Done** here.
4. Epic rollup to Done only when all children are terminal.

On failure: leave In Progress or Blocked; comment; do not Done.

## `/solve all` guard

Before draining: list In Progress on the project. If foreign live claims exist, do **not** start `/solve all` / `/solve all fast`. Use `/solve N` / explicit ids on unclaimed leaves only. One drain orchestrator per project.

## `/prb`

- Phase 2: PR comment only. Optional In Review. **Not Done.**
- After merge to `main` (explicit user approval): **Done** + production ship comment.
- Ignore / do not close foreign In Progress ids.

## `/issues`

File unassigned Backlog/Todo. Never assign as part of filing.
