---
name: start
description: >
  Use when the user runs /start, says "bootstrap a new project", "new site from
  the next starter", "spin up a new Next app", "start a project from the
  template", "validate this starter repo", or wants a greenfield product
  scaffolded from next-starter-template — or /start inside an existing checkout
  to validate bones plus AGENTS.md / VISION.md / README and repair onboard.
argument-hint: "[brief…] [--slug SLUG] [--dir PATH] [--team TEAM] [--docs-only|--no-linear|--no-build|--draft] [fast]"
---

# /start — Scaffold or validate, onboard, file V1, build

Two modes:

| Mode | When | What |
|------|------|------|
| **greenfield** | empty DEST | Clone **next-starter-template**, onboard docs, file V1, build on `dev` |
| **existing** | DEST is already a repo/tree | **Do not overlay the template.** Validate starter **bones** + `AGENTS.md` / `VISION.md` / `README.md` / identity. Repair docs. Then file V1 and build if still needed |

This is not `/issues` (tickets only) and not `/solve` alone.
**Greenfield** creates the repo. **Existing** must pass
[`references/existing-repo.md`](references/existing-repo.md) before later phases.

**North star:** after one `/start`, a stranger can open `$HOME/src/<slug>`, read
`VISION.md` + `AGENTS.md`, find `.WCP/issues/`, and run the app’s V1 on
local `dev`.

## Operating contract

- **Template:** [github.com/davidsolheim/next-starter-template](https://github.com/davidsolheim/next-starter-template). Do not invent a different stack. Do not mutate the template repo.
- **Onboard:** greenfield always runs DEST `AGENTS.md` first-run after scaffold. Existing runs the same protocol only for **repair** gaps (marker still present, missing/wrong VISION/AGENTS/README/identity). Questionnaire SoT is DEST `AGENTS.md` — do not fork the question list into this skill.
- **Queue default on:** write V1 leaves into `.WCP/issues/open/` ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Do not create a Linear project.
- **Build default on:** nested `/solve all` of the V1 leaves in DEST. Do not implement V1 yourself; do not skip build unless `--no-build` / `--docs-only` / `--draft`. A Notion failure does not skip the build when the files exist.
- **V1 = VISION.md “V1” section**, not the whole future product. Starter auth/CMS/admin/contact already exist — do not re-ticket them unless they must change for this product.
- **Git (greenfield):** fresh history (no template commits). `main` + lowercase `dev`. **Existing:** keep history; create `dev` if missing. No push/PR/deploy unless the user asks.
- **Secrets:** Doppler names only. Never reuse the starter Doppler project or another product’s `DATABASE_URL`. Never commit `.env` values.
- **Queue:** write issues under `.WCP/issues/` ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)), then upsert Notion ([`../docs/notion-issues.md`](../docs/notion-issues.md)). Do not call Linear. `--no-linear` skips filing and the Notion upsert.
- **Models:** every `spawn_subagent` sets `model: grok-4.6` ([`../docs/grok-models.md`](../docs/grok-models.md)).
- **Onboard questions:** required by DEST `AGENTS.md` first-run when that marker is present or docs fail validate. Prefill from the brief, flags, and existing README/VISION. Ask only gaps. Never product-onboard the public template repo.

## Trigger phrases

`/start`, `bootstrap a new project`, `new site from the next starter`, `spin up a new Next app`, `start a project from the template`, `scaffold from next-starter-template`, `validate this starter repo`, `onboard this existing repo`

## Invocation

```text
/start
/start a private claims network for shops
/start Acme App — merchant-to-merchant claim files
/start --slug acme-app brochure site for …
/start --dir $HOME/src/foo-com …
/start --docs-only …
/start --no-linear …
/start --no-build …
/start --draft …
/start fast …
```

| Arg | Meaning |
|-----|---------|
| free text | Product brief (name, audience, V1) |
| `--slug SLUG` | Repo / Doppler / package name (`acme-app`) |
| `--dir PATH` | Destination directory (default `$HOME/src/<slug>`) |
| `--team TEAM` | Linear team (else heuristic) |
| `--docs-only` | Greenfield: scaffold + onboard. Existing: validate + repair docs. Stop (no Linear, no build) |
| `--no-linear` | Skip Linear; still build from `VISION.md` V1 if not `--no-build` |
| `--no-build` | Stop after docs (+ Linear unless disabled) |
| `--draft` | Full plan + drafted Linear bodies in chat; do not create Linear; do not build |
| `fast` / `--fast` | No-op. Nested `/solve all` is already parallel |

### Parse order

1. `--dir PATH`, `--slug SLUG`, `--team TEAM`
2. `--docs-only` / `--no-linear` / `--no-build` / `--draft`
3. `fast` / `--fast` → `FAST_BUILD`
4. Remainder = `BRIEF`

`--docs-only` implies no Linear and no build. `--draft` implies no Linear create and no build.

## Skill paths

```text
START_SKILL_DIR = directory containing this SKILL.md
SCAFFOLD_MD     = $START_SKILL_DIR/references/scaffold.md
EXISTING_MD     = $START_SKILL_DIR/references/existing-repo.md
ONBOARD_MD      = $START_SKILL_DIR/references/onboard-docs.md
LINEAR_V1_MD    = $START_SKILL_DIR/references/linear-v1.md
HANDOFF_MD      = $START_SKILL_DIR/references/handoff.md
ISSUE_SKILL_MD  = $START_SKILL_DIR/../issue/SKILL.md
ISSUES_SKILL_MD = $START_SKILL_DIR/../issues/SKILL.md
SOLVE_SKILL_MD  = $START_SKILL_DIR/../solve/SKILL.md
```

Read `SCAFFOLD_MD` (greenfield), `EXISTING_MD` (existing), `ONBOARD_MD`, `LINEAR_V1_MD` before the matching phase.
Read `$ISSUE_SKILL_MD` Phase 1 + create gate, `$ISSUES_SKILL_MD` decomposition,
and `$SOLVE_SKILL_MD` only when that phase runs.

---

## Phase 0 — Capture the brief + mode

1. Parse args. Initialize `RUN_ID`.
2. Set **`MODE`**:
   - `existing` if `--dir` points at a **non-empty** tree, **or** cwd is a git/app repo (has `.git` or `package.json`) and is not `$HOME` and not the public template, and the user did not name a **different** empty slug dest
   - else `greenfield`
3. Prefill first-run fields from `BRIEF` + flags. **Existing:** also prefill from DEST `README.md`, `VISION.md` / `vision.md`, `package.json` `name`, `doppler.yaml`, `.linear-project`.
4. Greenfield: if name **and** slug are both missing, ask those two before DEST can be chosen. Infer slug from name (`Foo Bar` → `foobar-com` only when a domain is stated; else kebab-case).
5. Existing: empty `BRIEF` is OK. Do not ask “What are we starting?” if the tree already names the product.

Greenfield + empty brief + no slug → one question, “What are we starting?”
Existing + public template tree → stop (template maintenance, not product `/start`).

---

## Phase 1 — Destination + Linear team (no file writes yet)

**Destination**

- **Existing:** DEST = `--dir` if set, else cwd. Do **not** refuse because the tree is non-empty.
- **Greenfield:** [`references/scaffold.md`](references/scaffold.md) *Where*. Default `$HOME/src/<slug>`. `--dir` wins if that path is **empty or missing**.
- **Refuse both modes** if DEST is the public `next-starter-template` working tree.
- Greenfield only: if the chosen path exists and is non-empty, **switch to `MODE=existing`** on that path (validate) instead of asking for a new folder — unless the user clearly wanted a **new** sibling project (then ask for another path).

The work queue is `.WCP/issues/` in DEST ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Do not resolve a team or a Linear project.

---

## Phase 2 — Scaffold **or** validate

All further edits use **DEST**. Do not write into the public template.

### Greenfield

Follow [`references/scaffold.md`](references/scaffold.md) end-to-end.

After: dest exists, fresh git `main`, identity files retargeted (`package.json`, `doppler.yaml`), `bun.lock` kept, `.git` is **not** the template’s, template remote gone.

Optional: `gh repo create` **private** under `$GH_USER/<slug>` (`GH_USER=$(gh api user --jq .login)`) only if `gh` is authenticated **and** the user asked to create a remote. Default: local only.

### Existing

Follow [`references/existing-repo.md`](references/existing-repo.md).

1. Run **bones** (hard) and **docs/identity** + **git** checks. Print the pass/repair/fail table.
2. Bones **fail** → stop. Do not rsync the template; do not delete the tree.
3. Bones **pass** → keep the tree; Phase 3 only **repairs** listed doc/identity/git gaps.
4. Do not retarget Doppler/package identity until Phase 3 has a confirmed slug.

---

## Phase 3 — Onboard or repair docs

**Authority:** DEST `AGENTS.md` first-run when that marker exists; else [`references/onboard-docs.md`](references/onboard-docs.md) shapes. Existing repair list comes from Phase 2.

### Greenfield, or existing with first-run marker

1. Prefill from Phase 0. Ask remaining required questions in **one** message. Wait if required fields are missing.
2. Write `VISION.md`, product `README.md`, `.linear-project`, identity retarget.
3. **Overwrite DEST `AGENTS.md` in full** with the onboarded shape. Marker and questionnaire **must be gone**.
4. Commit (`docs: onboard <name>`). Greenfield: on `main`, then lowercase `dev`. Existing: current integration branch / `dev` (create `dev` if missing). Do not commit unrelated dirty files.

### Existing, marker already gone

1. Repair only failed doc/identity/git checks (missing `VISION.md`, leftover `vision.md`, README still the template, AGENTS missing Linear/Product/slug, Doppler still `next-starter-template`, no `.linear-project`, no `dev`).
2. If AGENTS is a thin leftover without platform rules, overwrite with the onboarded shape (need the same required fields — ask gaps only).
3. If **every** doc/identity check passed: **do not rewrite**. Note `docs: already correct` and continue.
4. Commit repairs only when files changed.

Phase 3 is not done while the first-run marker remains, or while `VISION.md` is missing, or while listed repairs are unwritten.

---

## Phase 4 — Queue + Notion

Skip if `--docs-only`, `--no-linear`, or `--draft` (draft still produces full bodies in chat).

Follow [`references/linear-v1.md`](references/linear-v1.md). Write the files, then upsert Notion ([`../docs/notion-issues.md`](../docs/notion-issues.md)). The database title is the repo slug. Its description is the origin URL.

1. Decompose **VISION.md V1 only** using `/issues` decomposition. Investigate the **new** tree (starter paths are real).
2. File atomic leaves in `open/`. A hard dependency goes in `blocked/` with `reason`. No epic file. Unassigned.
3. Do not file “add Better Auth / Drizzle / CMS / Doppler” unless V1 must change that starter behavior.
4. Always include a **product identity** leaf if metadata/home/footer still say starter.
5. Upsert each Notion row at the file's status.

`--draft`: print the plan and full leaf bodies. Do not write files or Notion.

Notion failure: keep the files and the docs; say which rows failed. Build still runs from the files.

---

## Phase 5 — Local install (non-secret)

In DEST:

```bash
bun install
```

Doppler: if CLI exists, `doppler setup --project <slug> --config development` **only** when that Doppler project already exists or the user asked to create it. Never copy another product’s config. Never print secrets.

If `DATABASE_URL` is missing: continue. Code still ships; runtime proof for DB-backed leaves may **block** those leaves (honest In Progress), not invent a database.

---

## Phase 6 — Build V1

Skip if `--docs-only`, `--no-build`, `--draft`, or no filed leaves (and not `--no-linear` with a vision-only build).

**Do not implement V1 in this orchestrator.** Nested `/solve`:

```text
spawn_subagent
  subagent_type: general-purpose
  isolation: none
  cwd: DEST
  model: grok-4.6
```

Prompt must include:

- Read `$SOLVE_SKILL_MD` and follow it end-to-end
- Workspace is **DEST** (the new repo)
- Queue is `.WCP/issues/` in DEST. Do not call Linear. Do not set Notion `done`
- `SOLVE_COUNT_MODE = all`
- `SHARED_DEV` = true (`/solve all` is shared-dev parallel; `FAST_BUILD` / `fast` is a no-op)
- `FAST_MODE` = false (do **not** use worktrees)
- `SELECTION_PIN` = V1 leaf ids in filing order (do not refill from other projects)
- `RUN_ID` = this start run
- `claimed-by: start` with this `RUN_ID` is this run
- Git contract: local `dev`, WCP occupancy ([`../docs/wcp.md`](../docs/wcp.md)), no push unless the user asked
- Runtime proof: [`../docs/prove-it-works.md`](../docs/prove-it-works.md)

`/start` must not write application source while the nested solve runs.

If `--no-linear` but build requested: implement V1 from `VISION.md` on local `dev` in DEST. Follow [`../docs/wcp.md`](../docs/wcp.md) and skill `water-cooler-protocol` for every edit (name yourself, lease pre-existing files, release before tests). Do not use bundled `/implement`.

After drain: local files are `done` after review. Notion stays `in-review` until `/prb` or `/yeet`. Do not set Notion `done`. Do not `/prb`.

---

## Phase 7 — Handoff

Follow [`references/handoff.md`](references/handoff.md). Then **stop**.

---

## Anti-patterns

- Editing `next-starter-template` in place
- Rsync/copying the template over an existing product (`MODE=existing`)
- Treating a non-starter stack as a failed onboard instead of a hard fail
- Skipping the bones table on an existing repo
- Keeping template git history or `origin` pointing at the template (greenfield)
- Leaving README/AGENTS describing the starter as this product
- Leaving `vision.md` instead of `VISION.md`
- Leaving `<!-- first-run: starter-onboard -->` in DEST after Phase 3
- Forking a second onboard questionnaire instead of DEST `AGENTS.md`
- Skipping `VISION.md`
- Filing V1 into some other product’s checkout
- Re-ticketing starter auth/CMS/admin as if they were missing
- One mega “build the product” ticket
- Implementing V1 in the `/start` orchestrator instead of nested `/solve`
- Nested solve without `cwd: DEST` or `model: grok-4.6`
- `/solve all` with no `SELECTION_PIN` on a shared team board
- Reusing Doppler/Neon from the starter or a sibling product
- Push/PR/production deploy without being asked
- Marking Linear Done from `/start`
- Asking the user for stack (it is the starter)
- Pretending V1 is done when nested solve failed or runtime proof was skipped

## Red flags — stop

- About to `rm -rf` or rsync over a non-empty destination
- About to `/start` product-onboard the public template repo
- About to `doppler setup` against `next-starter-template`
- About to file on Platform “because that’s where eng goes”
- About to skip Linear and also skip a vision V1 list
- About to declare done after docs-only when the user did not pass `--docs-only`

---

## Relation to other skills

| Skill | Role |
|-------|------|
| **`/start`** | Greenfield scaffold **or** existing-repo bones/docs validate+repair → file V1 → build on local `dev` |
| `/issue` | One ticket on an **existing** repo |
| `/issues` | Many tickets; **no** scaffold, **no** implement |
| `/solve` | Nested consumer for V1 leaves |
| `/prb` / `/yeet` | Later ship; not this skill |
| `/project-review` | Audit an existing product |

```text
/start (empty dest)     → scaffold → AGENTS.md / VISION.md / README.md
/start (existing dest)  → bones + docs validate → repair gaps
       → V1 leaves in `.WCP/issues/` plus Notion
       → nested /solve all in DEST
       → later /prb when the user wants main
```
