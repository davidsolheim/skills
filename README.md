# skills

Public, portable **[Grok](https://x.ai/)** agent skills used by [David Solheim](https://davidsolheim.com) to take product work from a one-line idea all the way to a merged pull request—without baking private client names, personal paths, or org-specific Linear boards into the skill text.

| | |
|---|---|
| **Author** | [David Solheim](https://davidsolheim.com) |
| **Site** | [davidsolheim.com](https://davidsolheim.com) |
| **Repo** | [github.com/davidsolheim/skills](https://github.com/davidsolheim/skills) |
| **Audience** | Anyone running agent-driven engineering with Linear + git |

These packages are **sanitized for open use**: no client brands as required defaults, no home-directory install paths, no private monorepo package names. Linear team/project resolution is always driven by *your* repo (`AGENTS.md`, `.linear-project`, etc.).

---

## Why this exists

Most agent “prompts” either stay private or leak the author’s machine and clients. This repo publishes a **repeatable delivery loop** as installable skills:

1. **File** implementation-ready Linear tickets from a short description (or a whole backlog dump).
2. **Solve** unblocked tickets onto a long-lived local `dev` branch with an implement→review loop.
3. **Ship** `dev` → PR into `main`, babysit CI, optionally run **this repo’s** production migrations, then merge.

The skills assume a professional full-stack product shop—not a single demo app. They are stack-aware when *your* docs say so, but they do not hard-code one company’s board or hoster.

---

## The delivery loop

```text
  idea / bug report
        │
        ▼
   /issue   ──► one Linear ticket (deep code map + AC)
   /issues  ──► many tickets (optional epic / blockedBy)
        │
        ▼
   /solve   ──► implement on short-lived branch → merge local dev only
        │         (/solve N · /solve all · /solve all fast)
        ▼
   /prb     ──► push origin/dev → PR main←dev → babysit CI → migrate? → merge
```

| Skill | Role | Default git effect |
|-------|------|--------------------|
| [`issue/`](./issue/) | Rapid intake: one description → one Linear issue | None (no commit / PR) |
| [`issues/`](./issues/) | Bulk intake: dump → many atomic tickets | None |
| [`solve/`](./solve/) | Pick next unblocked leaf(s), implement + review, land on **local `dev`** | Local only |
| [`prb/`](./prb/) | Ship finished `dev` work to **origin** and merge to **main** when green | Push + PR + merge |

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

- **Linear MCP** for ticket lifecycle (`/issue`, `/issues`, `/solve` closeout comments).
- **Git** with a durable local `dev` integration branch; short-lived issue branches per ticket.
- **Implement → review** loop (bundled `/implement` or equivalent) under `/solve`.
- **Production migrations** discovered per repo at ship time (`/prb`)—never invent a stack, never `db:push` to prod by default.

### Platform supersession (multi-ticket runs)

`/solve all` and multi-issue batches build **batch guidance** so architectural direction wins over stale tickets. Common pattern: older open work still mentions a warehouse or BaaS (e.g. ClickHouse, Convex) while newer tickets and repo docs make **Neon/Postgres** (or another stack) canonical. Guidance reorders migrations first and re-scopes or skips obsolete leaves.

---

## Skill catalog

### `/issue` — investigate & file one ticket

**Path:** [`issue/`](./issue/)

Rapid-fire intake. One short human description → deep, read-only codebase investigation → one **implementation-ready** Linear issue (code map, acceptance criteria, verification, drift anchors). Does **not** implement or open PRs.

**Triggers:** `/issue`, “file this bug”, “log this issue”, “ticket this”…

**Needs:** Linear MCP · current git workspace · optional `.linear-project` / `AGENTS.md`

---

### `/issues` — research & file many tickets

**Path:** [`issues/`](./issues/)

Bulk intake for brain dumps and residual backlogs. Shared investigation, atomic **solve-ready** leaves, optional epic / `blockedBy` / `relatedTo`. Same quality bar as `/issue`.

**Useful flags:** `--draft`, `--plan-only`, `--no-epic`, `--epic "Title"`, `--max N`

**Triggers:** `/issues`, “break this into tickets”, “file this list”…

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

### `/prb` — push `dev`, PR to `main`, babysit, migrate, merge

**Path:** [`prb/`](./prb/)

Ship **already finished** session work:

1. Refresh `origin/main` into local `main` and merge into local `dev` (**hard rule before any push**).
2. Push `origin/dev`.
3. Open/reuse PR `main` ← `dev`.
4. Babysit CI/bots (default every **5** minutes for **15** minutes).
5. If the ship includes schema/data migrations, discover and run **this repo’s** production migrate path (see [`prb/references/db-migrations.md`](./prb/references/db-migrations.md)).
6. Merge when quiet + green + migrations done (unless `--no-merge`).

**Useful flags:** `--no-merge`, `--skip-migrations`, `--watch-minutes N`, `--interval-minutes M`

---

## Install

Each skill is a directory with a `SKILL.md` and optional `references/`.

### Option A — copy into your agent skills root

```bash
git clone https://github.com/davidsolheim/skills.git
# Example for Grok user skills (path varies by agent):
cp -R skills/issue skills/issues skills/solve skills/prb ~/.grok/skills/
```

### Option B — point the agent at this repo

Clone or submodule this repository and configure your agent’s skills path to include the package directories (or the repo root if your runtime scans recursively).

### Option C — install one skill only

```bash
cp -R skills/issue ~/.grok/skills/issue
```

After install, invoke skills by slash command (`/issue`, `/solve all`, `/prb`, …) or the trigger phrases in each `SKILL.md` frontmatter.

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
├── issue/
│   ├── SKILL.md
│   └── references/
├── issues/
│   ├── SKILL.md
│   └── references/
├── solve/
│   ├── SKILL.md
│   └── references/
└── prb/
    ├── SKILL.md
    └── references/
```

---

## Related

- **Portfolio & contact:** [davidsolheim.com](https://davidsolheim.com)
- **This repository:** [github.com/davidsolheim/skills](https://github.com/davidsolheim/skills)
- **xAI / Grok:** [x.ai](https://x.ai/)

---

## License & contributions

Published for reuse and adaptation. If you improve a skill, PRs that keep the packages **generic** (no private paths or client defaults) are welcome.

When filing issues against *this* repo, a one-line `.linear-project` and clear acceptance criteria help the same loop dogfood itself.
