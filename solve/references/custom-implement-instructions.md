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
3. **Patterns**: match existing route, UI, data/runtime, and test conventions in the repo **unless** the current issue’s supersession notes explicitly replace that pattern. Prefer existing helpers over new abstractions when not superseding.
4. **Repo rules**: follow root `AGENTS.md` / `CLAUDE.md` / `README.md` for this workspace (Linear tracking, package manager, validation matrix, no reintroducing removed systems).
5. **Secrets**: never commit `.env`, print Doppler/tokens/connection strings, or log secrets.
6. **Git while implementing**: stay on the issue branch created by `/solve`. Do not switch to `main` or `dev`. Do not push. Do not open PRs.
7. **Commits**: prefer **not** committing during the implement loop. Leave a clean, reviewable working tree (or only intentional WIP commits on the issue branch). The `/solve` orchestrator stages and commits after the review loop and verification pass.
8. **Files**: stage-worthy changes only for this issue (including intentional overrides of earlier work). Leave unrelated dirty files untouched.
9. **Summary**: always write the implement summary file requested by the orchestrator (paths changed, design decisions, supersession overrides, verification notes).

## Prefer

- Smallest complete change that meets **current** acceptance criteria (which may include a selective override of prior work)
- Existing runtime routes / data helpers / server helpers already used in the consumer repo (e.g. Neon or equivalent) when this issue is not superseding that pattern
- Unit tests for new logic when the issue or repo validation matrix expects them
- Issue id in any commit messages if you must commit mid-loop (e.g. `TEAM-123: …`)

## Avoid

- Expanding scope “while you’re here” (supersession is not a license for unrelated rewrites)
- Preserving earlier-ticket behavior that this issue explicitly supersedes
- Pushing or creating PRs
- Discarding unrelated user work (`git reset --hard`, force-checkout over dirty unrelated files)
- Marking Linear Done (orchestrator owns Linear closeout)
- Merging into `dev` (orchestrator owns merge after verify)

## Optional overrides (edit as needed)

- Default implement effort when the user does not pass `--effort`: **5** (maximum rigor — up to 3 generals + all specialists)
- Extra reviewer focus (always mention in summary if relevant): auth/gating, billing, UI parity across surfaces, SEO only when marketing/public routes change
- UI changes: note manual/browser smoke expectations in the summary when browser tooling is unavailable

## Per-run user args

If the user passes free-text after `/solve` (other than `--effort N`, `--concurrency N`, `fast`/`--fast`, or an issue id), treat it as additional constraints and prepend it under **User constraints for this run** in the implementer prompt.

## Batch / fast guidance (multi-issue)

When the orchestrator provides a **batch guidance** path (`guidance.md` from `/solve all`, `/solve N` with `N≥2`, or `/solve … fast` — sometimes still named `architecture.md`):

1. **Read that file fully** before coding. Treat **canonical/abandoned platforms**, **Shared contracts**, **Supersession**, **execution order notes**, **Tech intersections**, and **Conflict zones** as hard constraints.
2. If the original Linear ticket and the guidance disagree on **platform/stack**, **guidance wins**. Implement re-scoped AC when `action=rescope`.
3. Do **not** invent parallel APIs, schemas, env keys, or patterns that conflict with the guidance or with other issues listed there.
4. Do **not** add dependencies on **abandoned platforms** (e.g. ClickHouse client work when canonical is Neon).
5. Stay on the assigned issue branch (and worktree in fast mode). Do **not** merge to `dev`/`main`, push, open PRs, or update Linear state (**implementer** — orchestrator owns those). On Cursor cloud the orchestrator opens **one** PR into `dev` (never `main`).
6. Prefer the smallest change that meets this leaf’s **current** (possibly re-scoped) acceptance criteria while remaining compatible with shared contracts.
7. Document supersession/rescope compliance in the implement summary.
