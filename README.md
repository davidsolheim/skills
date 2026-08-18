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

1. **Discover** what’s missing, broken, or inconsistent (`/project-review`), or **file** tickets from a human description (`/issue` / `/issues`).
2. **Tidy** the Linear board (`/tidy`): thicken thin tickets, close shipped work, stamp a weekly cooldown.
3. **Identify** a small high-value batch of open tickets, upgrade thin ones, and wait for approve (`/identify`).
4. **Solve** the approved tickets onto a long-lived local `dev` branch with an implement→review loop.
5. **Ship** `dev` → **local code review gate** (closed-loop Linear fix if needed) → push → PR into `main`, babysit CI, optionally run **this repo’s** production migrations, then merge.

The skills assume a professional full-stack product shop—not a single demo app. They are stack-aware when *your* docs say so, but they do not hard-code one company’s board or hoster.

---

## The delivery loop

```text
  idea / bug report / “audit this product”
        │
        ├─► /project-review  ──► agent invents findings → many solve-ready tickets
        ├─► /issue           ──► one Linear ticket (deep code map + AC)
        └─► /issues          ──► many tickets from a human dump
        │
        ▼
   /tidy     ──► upgrade thin tickets · status if shipped · weekly stamp
        │         skip live claims · needs-you list at end
        ▼
   /identify ──► pick 2–4 high-value leaves → upgrade thin tickets → wait
        │         approve → claim → /solve 1 per ID (sequential)
        │         reject  → next-best 2–4
        ▼
   /solve   ──► implement on short-lived branch → merge local dev only
        │         (/solve N · /solve all · /solve all fast · or /solve 1 ID from identify)
        ▼
   /prb     ──► local review gate → (issue+solve loop) → push origin/dev
                → PR main←dev → babysit CI → migrate? → merge
```

| Skill | Role | Default git effect |
|-------|------|--------------------|
| [`project-review/`](./project-review/) | Agentic audit: invent findings, file atomic Linear queue | None (no commit / PR) |
| [`issue/`](./issue/) | Rapid intake: one description → one Linear issue | None (no commit / PR) |
| [`issues/`](./issues/) | Bulk intake: dump → many atomic tickets | None |
| [`tidy/`](./tidy/) | Board hygiene: upgrade, relations, rollup, high-confidence close, shipped status | None (Linear only) |
| [`identify/`](./identify/) | Human-approved batch: rank open leaves, upgrade thin tickets, claim on approve, start `/solve` | None until approve, then same as `/solve` |
| [`solve/`](./solve/) | Pick next unblocked leaf(s), implement + review, land on **local `dev`** | Local only |
| [`prb/`](./prb/) | Local review gate + closed-loop fix, then ship `dev` to **origin** and merge to **main** when green | Local fixes + push + PR + merge |

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
| Agent runtime | **Grok Build** / compatible agent CLIs with MCP + subagents |

### Agent & delivery tooling

- **Linear MCP** for ticket lifecycle (`/issue`, `/issues`, `/project-review`, `/tidy` hygiene + stamps, `/identify` upgrades + claims, `/solve` closeout comments).
- **Git** with a durable local `dev` integration branch; short-lived issue branches per ticket.
- **Implement → review** loop (bundled `/implement` or equivalent) under `/solve`.
- **Browser / preview URL** optional for `/project-review` live walks (agent-browser, Chrome DevTools, etc.).
- **Production migrations** discovered per repo at ship time (`/prb`)—never invent a stack, never `db:push` to prod by default.
- **Local code review gate** on `/prb` before push: closed-loop `/issue` + `/solve` on local `dev` until the ship set is clean.

### Platform supersession (multi-ticket runs)

`/solve all` and multi-issue batches build **batch guidance** so architectural direction wins over stale tickets. Common pattern: older open work still mentions a warehouse or BaaS (e.g. ClickHouse, Convex) while newer tickets and repo docs make **Neon/Postgres** (or another stack) canonical. Guidance reorders migrations first and re-scopes or skips obsolete leaves.

---

## Skill catalog

### `/project-review` — agentic discovery → solve-ready Linear queue

**Path:** [`project-review/`](./project-review/)

The agent **invents** findings (completeness, bugs, UI consistency, taste→concrete AC, a11y, edge cases, cross-feature consistency)—no human laundry list required. Files atomic **solve-ready** leaves (optional epic) that `/solve` can claim. Discovery and Linear publish only; never implements or ships.

| Mode | Behavior |
|------|----------|
| **`fast`** | High-signal single-agent pass; prefer P0/P1 on primary journeys |
| **`deep`** (default) | Exhaustive multi-agent inventory + workers → local `issue-candidates/` → one Linear publish |

**Useful flags:** `--draft`, `--file`, `--p0-p1-only`, `--include-p2`, `--no-epic`, `--concurrency N`, `--scope-only`, `--url URL`, surface paths

**Triggers:** `/project-review`, “review this project”, “quality pass”, “UI audit”, “bug hunt”, “find issues for solve”…

**Needs:** Linear MCP (unless `--draft`) · git workspace · optional live app URL · optional browser tools

---

### `/issue` — investigate & file one ticket

**Path:** [`issue/`](./issue/)

Rapid-fire intake. One short human description → deep, read-only codebase investigation → one **implementation-ready** Linear issue (code map, acceptance criteria, verification, drift anchors). Does **not** implement or open PRs.

**Triggers:** `/issue`, “file this bug”, “log this issue”, “ticket this”…

**Needs:** Linear MCP · current git workspace · optional `.linear-project` / `AGENTS.md`

---

### `/issues` — research & file many tickets

**Path:** [`issues/`](./issues/)

Bulk intake for brain dumps and residual backlogs. Shared investigation, atomic **solve-ready** leaves, optional epic / `blockedBy` / `relatedTo`. Same quality bar as `/issue`. Prefer `/project-review` when the agent must invent the finding list.

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
2. **Local Code Review Gate** on `origin/main...dev` (high-signal findings only; load `## Code Review Rules` from AGENTS.md). If actionable findings exist: create Linear issues via `/issue`, fix on local `dev` via `/solve`, re-review in a closed loop until clean (default max **5** cycles). **No push until clean** (unless `--skip-review`). See [`prb/references/local-code-review.md`](./prb/references/local-code-review.md).
3. Push `origin/dev`.
4. Open/reuse PR `main` ← `dev`.
5. Babysit CI/bots (default every **5** minutes for **15** minutes).
6. If the ship includes schema/data migrations, discover and run **this repo’s** production migrate path (see [`prb/references/db-migrations.md`](./prb/references/db-migrations.md)).
7. Merge when quiet + green + migrations done (unless `--no-merge`).

**Useful flags:** `--no-merge`, `--skip-migrations`, `--skip-review`, `--exhaustive-review` / `--no-exhaustive`, `--max-fix-cycles N`, `--watch-minutes N`, `--interval-minutes M`

---

## Install

Each skill is a directory with a `SKILL.md` and optional `references/`.

### Option A — copy into your agent skills root

```bash
git clone https://github.com/davidsolheim/skills.git
# Example for Grok user skills (path varies by agent):
cp -R skills/issue skills/issues skills/project-review skills/tidy skills/identify skills/solve skills/prb ~/.grok/skills/
```

### Option B — point the agent at this repo

Clone or submodule this repository and configure your agent’s skills path to include the package directories (or the repo root if your runtime scans recursively).

### Option C — install one skill only

```bash
cp -R skills/issue ~/.grok/skills/issue
```

After install, invoke skills by slash command (`/project-review fast`, `/issue`, `/tidy`, `/identify`, `/solve all`, `/prb`, …) or the trigger phrases in each `SKILL.md` frontmatter.

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
4. **Local `dev` first** — `/solve` stops at local integration; humans (or `/prb`) control origin and production.
5. **Discovery over invention** — especially migrations and hoster CLIs: read the project; do not invent `db:push` to prod.
6. **Agent-operable tickets** — intake skills write tickets another agent can implement after a light drift check.

---

## Layout

```text
.
├── README.md
├── .linear-project          # tracking for this skills repo itself (optional for consumers)
├── project-review/
│   ├── SKILL.md
│   └── references/          # deep mode, candidates, templates, …
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
├── solve/
│   ├── SKILL.md
│   └── references/
└── prb/
    ├── SKILL.md
    └── references/          # local-code-review, db-migrations
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
