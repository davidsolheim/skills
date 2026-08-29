# Custom implement instructions for /solve

These instructions are injected into every `/solve` implementer (and, where relevant, reviewer) prompt.
Edit this file to change how implementation behaves under `/solve`. They do **not** change standalone `/implement`.

## Always apply

1. **Scope**: satisfy only the **current** Linear issue’s acceptance criteria. No drive-by refactors, no unrelated cleanup.
2. **Supersession / selective override** (when the orchestrator or issue body declares it):
   - The **current issue is authoritative** for its override scope. Prefer its AC over older Done/open issues it supersedes.
   - You **may and should** change, replace, or delete code that previously satisfied a superseded ticket **inside the listed override scope**.
   - Outside that scope, keep prior behavior and do not expand the rewrite.
   - Do **not** re-assert discarded earlier AC as requirements for this change.
   - In the implement summary, list: supersedes ids, override scope, and what was changed/removed vs left alone.
3. **Patterns**: match existing route, UI, Neon/runtime, and test conventions in the repo **unless** the current issue’s supersession notes explicitly replace that pattern. Prefer existing helpers over new abstractions when not superseding.
4. **Repo rules**: follow root `AGENTS.md` / `CLAUDE.md` / `README.md` for this workspace (Linear tracking, package manager, validation matrix, no reintroducing removed systems).
5. **Secrets**: never commit `.env`, print Doppler/tokens/connection strings, or log secrets.
6. **Git while implementing**: stay on the issue branch created by `/solve`. Do not switch to `main` or `dev`. Do not push. Do not open PRs.
7. **Commits**: prefer **not** committing during construction. Leave a clean, reviewable working tree (or only intentional WIP commits on the issue branch). The `/solve` orchestrator stages and commits after verification. Scratch files under `$TMPDIR` are never staged.
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
- Issue id in any commit messages if you must commit mid-loop (e.g. `TW-123: …`)

## Avoid

- Expanding scope “while you’re here” (supersession is not a license for unrelated rewrites)
- Preserving earlier-ticket behavior that this issue explicitly supersedes
- Pushing or creating PRs
- Discarding unrelated user work (`git reset --hard`, force-checkout over dirty unrelated files)
- Marking Linear Done (orchestrator owns Linear closeout)
- Merging into `dev` (orchestrator owns merge after verify)
- Claiming construction complete without runtime proof when the change is in-scope ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md))
- Running bundled `/implement` until-zero-nits, or treating `/prb` nits as construction blockers

## Optional overrides (edit as needed)

- Inner review when the user does not pass `--effort`: auto from [`../../docs/intensity.md`](../../docs/intensity.md) (`none` on light/standard, `bugs-only` on heavy/critical). Never spawn 2–6 reviewers. Reasoning is **low** via `solve-implementer`.
- Extra reviewer focus (always mention in summary if relevant): auth/gating, billing, portal/dashboard parity, SEO only when marketing routes change
- UI: drive the route; “browser unavailable” is **not** a pass — see prove-it-works.md

## Per-run user args

If the user passes free-text after `/solve` (other than `--effort N`, `--concurrency N`, `fast`/`--fast`, or an issue id), treat it as additional constraints and prepend it under **User constraints for this run** in the implementer prompt.

## Batch / fast guidance (multi-issue)

When the orchestrator provides a **batch guidance** path (`guidance.md` from `/solve all`, `/solve N` with `N≥2`, or `/solve … fast` — sometimes still named `architecture.md`):

1. **Read that file fully** before coding. Treat **canonical/abandoned platforms**, **Shared contracts**, **Supersession**, **execution order notes**, **Tech intersections**, and **Conflict zones** as hard constraints.
2. If the original Linear ticket and the guidance disagree on **platform/stack**, **guidance wins**. Implement re-scoped AC when `action=rescope`.
3. Do **not** invent parallel APIs, schemas, env keys, or patterns that conflict with the guidance or with other issues listed there.
4. Do **not** add dependencies on **abandoned platforms** (e.g. ClickHouse client work when canonical is Neon).
5. Stay on the assigned issue branch (and worktree in fast mode). Do **not** merge to `dev`/`main`, push, open PRs, or update Linear state (orchestrator owns those).
6. Prefer the smallest change that meets this leaf’s **current** (possibly re-scoped) acceptance criteria while remaining compatible with shared contracts.
7. Document supersession/rescope compliance in the implement summary.
