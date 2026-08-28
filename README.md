# skills

Public, portable **[Grok](https://x.ai/)** agent skills used by [David Solheim](https://davidsolheim.com) to take product work from a one-line idea all the way to a merged pull request—without baking private client names, personal paths, or org-specific Linear boards into the skill text.

| | |
|---|---|
| **Author** | [David Solheim](https://davidsolheim.com) |
| **Site** | [davidsolheim.com](https://davidsolheim.com) |
| **X** | [@davidtsolheim](https://x.com/davidtsolheim) |
| **Repo** | [github.com/davidsolheim/skills](https://github.com/davidsolheim/skills) |
| **Audience** | Anyone running agent-driven engineering with Linear + git |

These packages are **sanitized for open use**: no client brands as required defaults, no home-directory install paths, no private monorepo package names. Linear team/project resolution is always driven by *your* repo (`AGENTS.md`, `.linear-project`, etc.).

---

## Why this exists

Most agent “prompts” either stay private or leak the author’s machine and clients. This repo publishes a **repeatable delivery loop** as installable skills:

1. **Start** a product from [next-starter-template](https://github.com/davidsolheim/next-starter-template) (`/start`): scaffold or validate, onboard docs, Linear V1, nested `/solve` on local `dev`.
2. **Discover** what’s missing, broken, or inconsistent (`/project-review`), **walk** every front-facing screen (`/walk`), or **file** tickets from a human description (`/issue` / `/issues`).
3. **Tidy** the Linear board (`/tidy`): thicken thin tickets, close shipped work, stamp a weekly cooldown. **Look** at the open board urgent-first with `/stat` (read-only; not a solve batch).
4. **Identify** a small high-value batch of open tickets, upgrade thin ones, and wait for approve (`/identify`).
5. **Solve** the approved tickets onto a long-lived local `dev` branch with an implement→review loop.
6. **Ship** with `/prb` (review gate, CI babysit, migrate, merge) or `/yeet` (immediate merge, no babysit; still runtime-proof in-scope ships).

The skills assume a professional full-stack product shop—not a single demo app. They are stack-aware when *your* docs say so, but they do not hard-code one company’s board or hoster.

---

## The delivery loop

```text
  idea / new product / “audit this product”
        │
        ├─► /start           ──► scaffold or validate starter → onboard → Linear V1
        │                          → nested /solve all on local `dev`
        ├─► /project-review  ──► agent invents findings → many solve-ready tickets
        ├─► /walk            ──► live UI click-through → every bug/idea/improvement
        ├─► /issue           ──► one Linear ticket (deep code map + AC)
        └─► /issues          ──► many tickets from a human dump
        │
        ▼
   /tidy     ──► upgrade thin tickets · status if shipped · weekly stamp
        │         skip live claims · needs-you list at end
        │
   /stat     ──► read-only open board, urgent → least (not a batch)
        ▼
   /identify ──► pick 2–4 high-value leaves → upgrade thin tickets → wait
        │         approve → claim → /solve 1 per ID (sequential)
        │         reject  → next-best 2–4
        ▼
   /solve   ──► implement on short-lived branch → merge local dev only
        │         (/solve N · /solve all · /solve all fast · or /solve 1 ID from identify)
        ▼
   /prb     ──► grok-4.6 panel (thoroughness/security/rules/challenge)
                → (issue+solve loop) → push origin/dev
                → PR main←dev → babysit CI → migrate? → merge
        or
   /yeet    ──► push origin/dev → PR main←dev → runtime proof
                → migrate? → merge now (no panel, no CI wait)
```

| Skill | Role | Default git effect |
|-------|------|--------------------|
| [`start/`](./start/) | Greenfield scaffold **or** existing-repo bones/docs validate+repair → Linear V1 → nested `/solve` | Local `main` + `dev` (no push unless asked) |
| [`project-review/`](./project-review/) | Agentic audit: invent findings, file atomic Linear queue | None (no commit / PR) |
| [`walk/`](./walk/) | Live front-facing UI walk: debug every screen, file every finding | None (no commit / PR) |
| [`issue/`](./issue/) | Rapid intake: one description → one Linear issue | None (no commit / PR) |
| [`issues/`](./issues/) | Bulk intake: dump → many atomic tickets | None |
| [`tidy/`](./tidy/) | Board hygiene: upgrade, relations, rollup, high-confidence close, shipped status | None (Linear only) |
| [`stat/`](./stat/) | Read-only briefing of the open Linear board, urgent → least | None |
| [`identify/`](./identify/) | Human-approved batch: rank open leaves, upgrade thin tickets, claim on approve, start `/solve` | None until approve, then same as `/solve` |
| [`solve/`](./solve/) | Pick next unblocked leaf(s), implement + review, land on **local `dev`** | Local only |
| [`prb/`](./prb/) | Four-agent grok-4.6 review gate + closed-loop fix, then ship `dev` to **origin** and merge to **main** when green | Local fixes + push + PR + merge |
| [`yeet/`](./yeet/) | Quick ship: push `dev`, PR into `main`, merge immediately (no panel / no CI wait). In-scope runtime proof still required | Push + PR + merge |

**Branch convention (all skills):** integration branch is always lowercase **`dev`**; trunk is **`main`**. Never capital-`D` `Dev`.

---

## Common technology stacks

These skills are intentionally portable, but they were shaped around the stacks David uses day-to-day for product and client work. Agents following the skills will **prefer patterns already in your repo**; the list below is the usual “happy path” when greenfield or when docs are thin.

### Application

| Layer | Typical choices |
|-------|-----------------|
| Language | **TypeScript** (strict), Node.js LTS |
| Framework | **Next.js** (App Router), **React** |
| UI | **Tailwind CSS**, **shadcn/ui**, accessible HTML primitives |
| Validation / API | Zod, server actions / route handlers, typed clients |
| Auth (when present) | Clerk, Auth0, or app-native session patterns—whatever the repo already uses |

### Data & backend

| Layer | Typical choices |
|-------|-----------------|
| Database | **Neon** (serverless Postgres), Postgres generally |
| ORM / SQL | Drizzle, Prisma, or raw SQL—**only as documented in the target repo** |
| Cache / jobs | Redis, Inngest, or platform crons when the project already has them |
| Files / media | Vercel Blob, S3-compatible storage, CDN-backed public assets |

### Platform & ops

| Layer | Typical choices |
|-------|-----------------|
| Hosting | **Vercel** (preview + production) |
| Secrets | **Doppler** (or Vercel env / `.env.local` for local only)—skills never commit secrets |
| CI | GitHub Actions + `gh` CLI for PRs, checks, and merge |
| Project tracking | **Linear** (MCP tools: list/create/update issues, comments, projects) |
| Agent runtime | **Grok Build**. Subagents are **Grok-only** (`grok-4.6` default, `grok-4.5` for explore fan-out) — see [`docs/grok-models.md`](./docs/grok-models.md) |

### Agent & delivery tooling

- **Linear MCP** for ticket lifecycle (`/start` product project + V1 epic, `/issue`, `/issues`, `/project-review`, `/walk` UI findings, `/tidy` hygiene + stamps, `/stat` read-only briefing, `/identify` upgrades + claims, `/solve` closeout comments).
- **Git** with a durable local `dev` integration branch; short-lived issue branches per ticket.
- **Implement → review** loop (bundled `/implement` or equivalent) under `/solve`.
- **Browser / preview URL** optional for `/project-review`; **required** for `/walk` (agent-browser, Chrome DevTools, etc.).
- **Production migrations** discovered per repo at ship time (`/prb`, `/yeet`)—never invent a stack, never `db:push` to prod by default.
- **Local code review gate** on `/prb` before push: four `grok-4.6` reviewers (thoroughness, security, rules, challenge), then closed-loop `/issue` + `/solve` on local `dev` until the ship set is clean.

### Platform supersession (multi-ticket runs)

`/solve all` and multi-issue batches build **batch guidance** so architectural direction wins over stale tickets. Common pattern: older open work still mentions a warehouse or BaaS (e.g. ClickHouse, Convex) while newer tickets and repo docs make **Neon/Postgres** (or another stack) canonical. Guidance reorders migrations first and re-scopes or skips obsolete leaves.

---

## Skill catalog

### `/start` — scaffold or validate, onboard, Linear V1, build on local `dev`

**Path:** [`start/`](./start/)

Greenfield: clone **[next-starter-template](https://github.com/davidsolheim/next-starter-template)** into a fresh tree (no template git history), onboard `AGENTS.md` / `VISION.md` / `README.md`, create a Linear project + V1 epic, then nested `/solve all` on local `dev`. Existing dest: **do not overlay the template** — validate starter bones, repair docs/identity, reuse the bound Linear project if present.

| Flag | Behavior |
|------|----------|
| `--slug SLUG` | Repo / Doppler / package name |
| `--dir PATH` | Destination (default `$HOME/src/<slug>`) |
| `--team TEAM` | Linear team (else dest docs / `list_teams`) |
| `--docs-only` | Scaffold or repair docs; no Linear, no build |
| `--no-linear` | Skip Linear; still build from `VISION.md` V1 |
| `--no-build` | Stop after docs (+ Linear unless disabled) |
| `--draft` | Print project + leaf bodies; do not create Linear; do not build |
| `fast` / `--fast` | Nested `/solve all fast` for V1 |

**Triggers:** `/start`, “bootstrap a new project”, “spin up a new Next app”, “validate this starter repo”…

**Needs:** git · companion `/issue` `/issues` `/solve` · Linear MCP unless `--docs-only` / `--no-linear` / `--draft` · optional `gh` + Doppler

**Companion refs:** [`start/references/scaffold.md`](./start/references/scaffold.md), [`start/references/existing-repo.md`](./start/references/existing-repo.md), [`start/references/onboard-docs.md`](./start/references/onboard-docs.md), [`start/references/linear-v1.md`](./start/references/linear-v1.md)

Default dest is **local only**. No GitHub remote, push, or production deploy unless you ask. Nested solve pins **this product’s** V1 leaves — it does not drain some other board.

---

### `/project-review` — agentic discovery → solve-ready Linear queue

**Path:** [`project-review/`](./project-review/)

The agent **invents** findings (completeness, bugs, UI consistency, taste→concrete AC, a11y, edge cases, cross-feature consistency)—no human laundry list required. Files atomic **solve-ready** leaves (optional epic) that `/solve` can claim. Discovery and Linear publish only; never implements or ships.

| Mode | Behavior |
|------|----------|
| **`fast`** | High-signal single-agent pass; prefer P0/P1 on primary journeys |
| **`deep`** (default) | Exhaustive multi-agent inventory + workers → local `issue-candidates/` → one Linear publish |

**Useful flags:** `--draft`, `--file`, `--p0-p1-only`, `--include-p2`, `--no-epic`, `--concurrency N`, `--scope-only`, `--url URL`, surface paths

**Triggers:** `/project-review`, “review this project”, “quality pass”, “bug hunt”, “find issues for solve”…

**Needs:** Linear MCP (unless `--draft`) · git workspace · optional live app URL · optional browser tools

A **live click-through of every front-facing screen** is **`/walk`**, not this skill.

---

### `/walk` — live front-facing UI walk → Linear queue

**Path:** [`walk/`](./walk/)

The agent **drives the live app** like a user: every front-facing route, click/type/submit, console and network debug, desktop + mobile on primary journeys. Files **every** actionable **bug, idea, and improvement** as atomic solve-ready Linear leaves (optional epic). Discovery and Linear publish only; never implements or ships. Browser tools are **required** — a code-only skim is a failed walk.

| Flag | Behavior |
|------|----------|
| `--url URL` | Preview / live origin to drive |
| `--draft` | Local candidates only; do not create Linear issues |
| `--signed-out` / `--signed-in` | Auth scope (default: both when login is possible) |
| `--no-epic` | Flat leaves |
| `--desktop-only` / `--mobile-only` | Skip the other viewport sample |
| free text | Surface bias — still finish the rest of the inventory |

**Triggers:** `/walk`, “walk the UI”, “click through the app”, “walk every screen”, “front-facing UI audit”…

**Needs:** Linear MCP (unless `--draft`) · git workspace · reachable live/local app · a browser driver that can click and read console

**Companion refs:** [`walk/references/walk-protocol.md`](./walk/references/walk-protocol.md), [`walk/references/finding-kinds.md`](./walk/references/finding-kinds.md)

---

### `/issue` — investigate & file one ticket

**Path:** [`issue/`](./issue/)

Rapid-fire intake. One short human description → deep, read-only codebase investigation → one **implementation-ready** Linear issue (code map, acceptance criteria, verification, drift anchors). Does **not** implement or open PRs.

**Triggers:** `/issue`, “file this bug”, “log this issue”, “ticket this”…

**Needs:** Linear MCP · current git workspace · optional `.linear-project` / `AGENTS.md`

---

### `/issues` — research & file many tickets

**Path:** [`issues/`](./issues/)

Bulk intake for brain dumps and residual backlogs. Shared investigation, atomic **solve-ready** leaves, optional epic / `blockedBy` / `relatedTo`. Same quality bar as `/issue`. Prefer `/project-review` or `/walk` when the agent must invent the finding list.

**Useful flags:** `--draft`, `--plan-only`, `--no-epic`, `--epic "Title"`, `--max N`

**Triggers:** `/issues`, “break this into tickets”, “file this list”…

---

### `/tidy` — Linear board hygiene (weekly cooldown)

**Path:** [`tidy/`](./tidy/)

Full pass on the current project’s **due** issues. Thickens thin tickets to the `/issue` bar, retitles, fixes obvious relations, rolls up finished epics, applies **high-confidence** Duplicate/Canceled, and sets status when work is already shipped (**In Review** on `origin/dev`, **Done** only on `main`). Skips live foreign claims. Finishes the board, then one **needs-you** list.

| Invocation | Behavior |
|------------|----------|
| `/tidy` | Every issue whose last tidy-pass is ≥ 7 days ago (or never) |
| `/tidy --force` | Ignore cooldown for the whole board |
| `/tidy TEAM-123` | That issue only; ignore its cooldown |

Last pass is a parseable Linear `tidy-pass:` comment plus a machine-local ledger under `~/.grok/tidy/ledgers/`. Does **not** implement or claim.

**Triggers:** `/tidy`, “tidy Linear”, “clean up the board”, “thicken thin tickets”, “close issues that are already done”

**Needs:** Linear MCP · current git workspace · companion `/issue` (quality bar)

**Companion refs:** [`tidy/references/ledger.md`](./tidy/references/ledger.md), [`tidy/references/actions.md`](./tidy/references/actions.md)

---

### `/stat` — open board, urgent first (read-only)

**Path:** [`stat/`](./stat/)

Lists **every issue that still needs resolution** on this repo’s Linear project, sorted **most urgent → least**. Includes blocked, claimed, In Progress, In Review, and epic shells. Does **not** implement, claim, tidy, upgrade, or start `/solve` / `/identify`.

| Invocation | Behavior |
|------------|----------|
| `/stat` | Full open board |
| `/stat N` | Top **N** after sort; still reports full counts |
| `/stat checkout` | Area filter on title / labels / identifier |

**Triggers:** `/stat`, “board status”, “what's open”, “what's left to solve”

**Needs:** Linear MCP · current git workspace · companion `/issue` (team/project) and `/identify` ranking keys

Do **not** use for session `/status` (auth/model/context) or for picking a solve batch (`/identify`).

---

### `/identify` — recommend a 2–4 ticket batch, then `/solve` on approve

**Path:** [`identify/`](./identify/)

Reads the current project’s **open** Linear board, keeps only `/solve`-eligible unblocked **leaves**, ranks by **Linear priority then user-facing impact**, and proposes a **small mix** (2–4). Thin tickets in that set are rewritten in Linear to the `/issue` bar **before** you see them. The skill **stops and waits**.

| You say | What happens |
|---------|----------------|
| **Approve** | Claim each id (`claimed-by:` + In Progress), then sequential `/solve 1 <ID>` in the proposed order |
| **Reject** | Those ids are excluded for the session; next-best 2–4 is proposed |

Plain `/identify` only — no size, theme, or id args. `/solve` still works without Identify when you want auto-pick.

**Triggers:** `/identify`, “identify work”, “pick a batch”, “what should we solve”, “recommend issues to solve”

**Needs:** Linear MCP · current git workspace · companion `/issue` (upgrades) and `/solve` (eligibility + implement)

**Companion refs:** [`identify/references/ranking.md`](./identify/references/ranking.md), [`identify/references/upgrade.md`](./identify/references/upgrade.md)

---

### `/solve` — implement unblocked Linear work onto local `dev`

**Path:** [`solve/`](./solve/)

Selects the next unblocked leaf (or drains the board), runs the full implement→review loop, verifies, commits on a short-lived branch, and **merges into local `dev` only**. Does not push or open a PR unless you ask.

| Invocation | Behavior |
|------------|----------|
| `/solve` | One issue (default) |
| `/solve N` | Up to N issues |
| `/solve all` | Drain every eligible unblocked leaf (no soft cap) |
| `/solve all fast` | Parallel worktree workers after shared batch guidance |

**Also supports:** preferred issue id, `--effort N`, `fast`, `--concurrency M`

**Companion refs:** batch guidance, fast mode, git-dev workflow, custom implement instructions (all under `solve/references/`).

---

### `/prb` — local review gate, push `dev`, PR to `main`, babysit, migrate, merge

**Path:** [`prb/`](./prb/)

Ship **already finished** session work:

1. Refresh `origin/main` into local `main` and merge into local `dev` (**hard rule before any push**).
2. **Local Code Review Gate** on `origin/main...dev`: four parallel `grok-4.6` reviewers (thoroughness, security, rules, challenge), merged through [`prb/references/review-rubric.md`](./prb/references/review-rubric.md). High-signal findings only; load `## Code Review Rules` from AGENTS.md. If actionable findings exist: create Linear issues via `/issue`, fix on local `dev` via `/solve`, re-run the panel until clean (default max **5** cycles). **No push until clean** (unless `--skip-review`). See [`prb/references/local-code-review.md`](./prb/references/local-code-review.md).
3. Push `origin/dev`.
4. Open/reuse PR `main` ← `dev`.
5. Babysit CI/bots (default every **5** minutes for **15** minutes).
6. If the ship includes schema/data migrations, discover and run **this repo’s** production migrate path (see [`prb/references/db-migrations.md`](./prb/references/db-migrations.md)).
7. Merge when quiet + green + migrations done (unless `--no-merge`).

**Useful flags:** `--no-merge`, `--skip-migrations`, `--skip-review`, `--exhaustive-review` / `--no-exhaustive`, `--max-fix-cycles N`, `--watch-minutes N`, `--interval-minutes M`

---

### `/yeet` — quick ship `dev` → `main` (no babysit)

**Path:** [`yeet/`](./yeet/)

Ship **already finished** session work **now**. `/prb` is the careful path. `/yeet` skips the review panel and CI wait.

1. Refresh `origin/main` into local `main` and `dev`.
2. Commit leftover session files on `dev` if needed.
3. Push `origin/dev`, open/reuse PR `main` ← `dev`.
4. **Runtime proof** for in-scope ships ([`docs/prove-it-works.md`](./docs/prove-it-works.md)) — skip the panel, **not** the proof.
5. Additive production migrate when the ship includes migrations.
6. Merge immediately (`gh pr merge --rebase --admin`). Do **not** re-push `dev` after merge.
7. Linear comment + **Done** when ticket ids are known.

**Useful flags:** `--via-dev-pr`, `--skip-migrations`, `--no-commit`

**Triggers:** `/yeet`, “yeet it”. Do **not** steal `/prb` when the user asked to babysit or carefully ship.

**Needs:** `gh` auth · git workspace with long-lived `dev` · companion `/prb` migrate discovery

---

## Install

Each skill is a directory with a `SKILL.md` and optional `references/`.

### Option A — copy into your agent skills root

```bash
git clone https://github.com/davidsolheim/skills.git
# Example for Grok user skills (path varies by agent):
cp -R skills/start skills/issue skills/issues skills/project-review skills/walk skills/tidy skills/stat skills/identify skills/solve skills/prb skills/yeet ~/.grok/skills/
```

### Option B — point the agent at this repo

Clone or submodule this repository and configure your agent’s skills path to include the package directories (or the repo root if your runtime scans recursively).

### Option C — install one skill only

```bash
cp -R skills/issue ~/.grok/skills/issue
```

After install, invoke skills by slash command (`/start`, `/project-review fast`, `/walk`, `/issue`, `/tidy`, `/stat`, `/identify`, `/solve all`, `/prb`, `/yeet`, …) or the trigger phrases in each `SKILL.md` frontmatter.

---

## Repo configuration tips

Skills auto-resolve Linear **team** and **project** from the target codebase. Put the truth in the consumer repo, not in this skill pack:

| File | Purpose |
|------|---------|
| `AGENTS.md` / `CLAUDE.md` | Linear team, project, verify commands, deploy notes |
| `.linear-project` | Single line: project name or slug |
| `.linear.json` / yaml | Optional structured Linear config |
| `README.md` | Branching, migrate scripts, CI expectations |

Example `.linear-project`:

```text
My Product Launch
```

**Secrets:** never put Doppler values, connection strings, or API tokens in Linear descriptions or in commits. Skills only use **names** of env vars and documented migrate scripts.

---

## Design principles

1. **Portable** — work from any skills install path; references are skill-relative (`references/…` or `$SOLVE_SKILL_DIR`).
2. **Repo-driven** — Linear boards, migrate commands, and stack choices come from the **consumer** repo.
3. **No PII / no private clients** — public packages must not require inventRight, personal home paths, or internal product codenames.
4. **Local `dev` first** — `/solve` stops at local integration; humans (or `/prb` / `/yeet`) control origin and production.
5. **Discovery over invention** — especially migrations and hoster CLIs: read the project; do not invent `db:push` to prod.
6. **Agent-operable tickets** — intake skills write tickets another agent can implement after a light drift check.
7. **Grok-only models** — every `spawn_subagent` passes `model: grok-4.6` (or `grok-4.5` for read-only explore). Never inherit the host default. Never Claude/GPT/Composer.

---

## Layout

```text
.
├── README.md
├── .linear-project          # tracking for this skills repo itself (optional for consumers)
├── start/
│   ├── SKILL.md
│   └── references/          # scaffold, existing-repo, onboard, linear-v1, handoff
├── project-review/
│   ├── SKILL.md
│   └── references/          # deep mode, candidates, templates, …
├── walk/
│   ├── SKILL.md
│   └── references/          # live UI protocol, finding kinds, handoff
├── issue/
│   ├── SKILL.md
│   └── references/
├── issues/
│   ├── SKILL.md
│   └── references/
├── identify/
│   ├── SKILL.md
│   └── references/          # ranking, upgrade
├── tidy/
│   ├── SKILL.md
│   └── references/          # ledger, actions
├── stat/
│   └── SKILL.md             # read-only open board, urgent first
├── solve/
│   ├── SKILL.md
│   └── references/
├── prb/
│   ├── SKILL.md
│   └── references/          # local-code-review, review-rubric, reviewer-prompts, db-migrations
├── yeet/
│   └── SKILL.md             # quick ship; shares prove-it-works + prb migrate discovery
└── docs/
    ├── prove-it-works.md
    ├── grok-models.md
    ├── linear-comments.md
    └── rfc-multiplayer-linear.md
```

---

## Related

- **Portfolio & contact:** [davidsolheim.com](https://davidsolheim.com)
- **X:** [@davidtsolheim](https://x.com/davidtsolheim)
- **This repository:** [github.com/davidsolheim/skills](https://github.com/davidsolheim/skills)
- **xAI / Grok:** [x.ai](https://x.ai/)

---

## License & contributions

Published for reuse and adaptation. If you improve a skill, PRs that keep the packages **generic** (no private paths or client defaults) are welcome.

When filing issues against *this* repo, a one-line `.linear-project` and clear acceptance criteria help the same loop dogfood itself.
