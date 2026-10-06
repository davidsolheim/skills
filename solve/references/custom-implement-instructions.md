# Custom implement instructions for /solve

These instructions are injected into every `/solve` implementer (and, where relevant, reviewer) prompt.
Edit this file to change how implementation behaves under `/solve`. They do **not** change standalone `/implement`.

## Always apply

1. **Scope**: satisfy only the **current** issue file’s acceptance criteria. No drive-by refactors, no unrelated cleanup.
2. **Supersession / selective override** (when the orchestrator or issue body declares it):
   - The **current issue is authoritative** for its override scope. Prefer its AC over older Done/open issues it supersedes.
   - You **may and should** change, replace, or delete code that previously satisfied a superseded ticket **inside the listed override scope**.
   - Outside that scope, keep prior behavior and do not expand the rewrite.
   - Do **not** re-assert discarded earlier AC as requirements for this change.
   - In the implement summary, list: supersedes ids, override scope, and what was changed/removed vs left alone.
3. **Patterns**: match existing route, UI, Neon/runtime, and test conventions in the repo **unless** the current issue’s supersession notes explicitly replace that pattern. Prefer existing helpers over new abstractions when not superseding.
4. **Repo rules**: follow root `AGENTS.md` / `CLAUDE.md` / `README.md` for this workspace (`.wcp/issues/` tracking, package manager, validation matrix, no reintroducing removed systems).
5. **Secrets**: never commit `.env`, print Doppler/tokens/connection strings, or log secrets.
6. **Git while implementing**: stay on the branch the orchestrator assigned. **Shared-dev (default parallel):** stay on local `dev`; do not create branches, worktrees, stash, reset, or restore sibling files; do not commit. **Sequential and worktree:** stay on the issue branch; do not switch to `main`; do not commit and do not stash. Never push or open PRs.
6b. **Occupancy (wcp)**: load skill `water-cooler-protocol` and [`../../docs/wcp.md`](../../docs/wcp.md). Read `.wcp/issues/in-progress/` and work around the paths those issues list. Add each path you write to this issue's `files`. There is no `wcp` command. Never rewind sibling edits. If a file already has uncommitted changes, read the `in-progress/` or `done/` issue that lists that path and the other paths in its `files`. Keep the behavior its `acceptance` describes.
7. **Commits**: do not commit and do not stash. Leave the tree dirty. The `/solve` orchestrator commits after verification, and only when `.wcp/issues/in-progress/` is empty. Scratch files under `$TMPDIR` are never staged.
8. **Files**: stage-worthy changes only for this issue (including intentional overrides of earlier work). Leave unrelated dirty files untouched.
9. **Summary**: always write the implement summary file requested by the orchestrator (paths changed, design decisions, supersession overrides, verification notes, **runtime-proof evidence**).
10. **Runtime proof**: follow [`../../docs/prove-it-works.md`](../../docs/prove-it-works.md). Do not claim construction complete on typecheck/tests/build alone when the change is user-visible, auth, billing, public API, schema, or a shared helper.
11. **Boundaries**: parse/validate at HTTP, env, webhooks, and external JSON. After that, trust internal types. Do not nil-guard a crash and call it fixed.
12. **Models:** every `spawn_subagent` (implementer, reviewers, nested workers) must pass `model: grok-4.6` per [`../../docs/grok-models.md`](../../docs/grok-models.md). Do not inherit the parent. Do not spawn Claude, GPT, Gemini, Composer, or Cursor Auto.

## Ticket kind

| The issue is… | While implementing |
|---------------|-------------------|
| Bug with a cheap local test path | Write the failing test first, then the fix |
| User-visible UI | Match the named sibling/reference; smallest change; delight over convenience |
| Crosses a module boundary | Settle types and call shape before filling logic |
| Shared helper / schema / auth | Prove the one safety fact by running code |

## Prefer

- Smallest complete change that meets **current** acceptance criteria (which may include a selective override of prior work). Delete dead weight in-scope before adding.
- Existing Neon runtime routes / `useRuntimeResource` / server helpers when the repo already uses them **and** this issue is not superseding that pattern
- Unit tests for new logic when the issue or repo validation matrix expects them; **failing test first** for bugs when a cheap local path exists
- The orchestrator puts the issue id in the commit subject (`0123: …`). Workers do not commit mid-loop.

## Avoid

- Expanding scope “while you’re here” (supersession is not a license for unrelated rewrites)
- Preserving earlier-ticket behavior that this issue explicitly supersedes
- Pushing or creating PRs
- Discarding unrelated user work (`git reset --hard`, force-checkout over dirty unrelated files)
- Setting file `done` or Notion `done` (reviewer sets file `done`; orchestrator writes `commit`; `/prb` sets Notion `done`)
- Merging into `dev` or committing on `dev` (orchestrator owns commit after verify)
- Claiming construction complete without runtime proof when the change is in-scope ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md))
- Running bundled `/implement` until-zero-nits, or treating `/prb` nits as construction blockers

## Optional overrides (edit as needed)

- Inner review when the user does not pass `--effort`: auto from [`../../docs/intensity.md`](../../docs/intensity.md) (`none` on light/standard, `bugs-only` on heavy/critical). Never spawn 2–6 reviewers. Reasoning is **low** via `solve-implementer`.
- Extra reviewer focus (always mention in summary if relevant): auth/gating, billing, portal/dashboard parity, SEO only when marketing routes change
- UI: drive the route; “browser unavailable” is **not** a pass — see prove-it-works.md

## Per-run user args

If the user passes free-text after `/solve` (other than `--effort N`, `--concurrency N`, `fast`/`--fast`, or an issue id), treat it as additional constraints and prepend it under **User constraints for this run** in the implementer prompt.

## Batch / fast guidance (multi-issue)

When the orchestrator provides a **batch guidance** path (`guidance.md` from `/solve all`, `/solve N` with `N≥2`, or any parallel `/solve` — sometimes still named `architecture.md`):

1. **Read that file fully** before coding. Treat **canonical/abandoned platforms**, **Shared contracts**, **Supersession**, **execution order notes**, **Tech intersections**, and **Conflict zones** as hard constraints.
2. If the original issue file and the guidance disagree on **platform/stack**, **guidance wins**. Implement re-scoped AC when `action=rescope`.
3. Do **not** invent parallel APIs, schemas, env keys, or patterns that conflict with the guidance or with other issues listed there.
4. Do **not** add dependencies on **abandoned platforms** (e.g. ClickHouse client work when canonical is Neon).
5. Stay on the assigned branch. Shared-dev: stay on `dev`. Read `in-progress/` and work around those paths. Do **not** commit. Do **not** merge to `dev`/`main`, push, or open PRs. When the writing is finished, move the issue to `done/`. Do not call Linear.
6. Prefer the smallest change that meets this leaf’s **current** (possibly re-scoped) acceptance criteria while remaining compatible with shared contracts.
7. Document supersession/rescope compliance in the implement summary.
