# /prb — Project-aware production DB migrations

When `/prb` ships work that includes schema/data migrations, **apply them carefully at the production gate** using **this repo’s** documented migration procedure. Do not invent a stack. Do not assume every app migrates the same way.

Vercel (and most CI here) often **does not** run migrations on deploy. Merging code that requires new tables/columns without a production migrate is a skill failure when the ship set includes migrations.

---

## 1. Detect whether this ship needs migrations

Run from the git root of the project `/prb` is operating on.

### 1A. Ship-set inventory

```bash
# Migration-related paths introduced or changed vs trunk
git diff --name-only origin/main...dev -- \
  '**/drizzle/**' \
  '**/prisma/migrations/**' \
  '**/supabase/migrations/**' \
  '**/packages/db/migrations/**' \
  '**/db/migrations/**' \
  '**/migrations/**' \
  '**/lib/db/migrate.*' \
  '**/drizzle.config.*' \
  '**/schema.prisma'
```

Also scan commit messages on `origin/main..dev` for `migrat`, `schema`, `drizzle`, `prisma`.

Record:

| Field | Value |
|-------|--------|
| `MIGRATIONS_IN_SHIP` | yes / no |
| `MIGRATION_PATHS` | list of changed migration/schema files |
| `MIGRATION_STACK` | discovered stack (below) or `none` |

If **no** migration paths and no project migrate script dependency in the ship, set `MIGRATIONS_IN_SHIP=no` and skip the rest of this reference (still mention “migrations: n/a” in the final report).

### 1B. Uncommitted / untracked migration files

If new migration SQL exists only in the working tree and is not on `dev`, **stop** — migrations must be committed on `dev` before production apply. Do not apply uncommitted SQL to production.

---

## 2. Discover **this project’s** migration procedure (required)

Read **in order** until you can fill the discovery table. Prefer the first clear source of truth.

1. `AGENTS.md` / `CLAUDE.md` / repo agent rules (DB / Doppler / migrate sections)
2. `docs/DATABASE_MIGRATIONS.md`, `docs/**/migrat*.md`, `docs/migration/**`
3. `README.md` (Database / Migration sections)
4. Root and workspace `package.json` scripts matching `db:migrate`, `db:migrate:*`, `migrate`, `prisma:migrate*`
5. Config files: `drizzle.config.*`, `prisma/schema.prisma`, `doppler.yaml`
6. Monorepo packages: `packages/db/**`, `apps/*/package.json` filter scripts

### Discovery table (fill every run when migrations apply)

| Field | How to resolve |
|-------|----------------|
| `PKG` | Package manager: `bun` if `bun.lock`/`bun.lockb`; else `pnpm` if `pnpm-lock.yaml`; else `npm`/`yarn` from lockfile |
| `MIGRATE_SCRIPT` | Prefer `db:migrate` over `db:push`. If `db:push` is the only option and docs forbid it for shared DBs, **stop** |
| `MIGRATE_CMD` | Exact command from docs/scripts (e.g. `bun run db:migrate`, `pnpm --dir packages/db …`) |
| `DOPPLER_PROJECT` | From `doppler.yaml` `setup.project`, AGENTS examples, or folder name only if documented |
| `DOPPLER_PRODUCTION` | Production config name: usually `production` or `prd` (sometimes `prod`). **Never guess** if both `prd` and `production` exist without docs — list configs and pick the one docs map to Vercel Production |
| `DOPPLER_PREVIEW` | Optional: `preview` / `stg` if docs say migrate preview too |
| `ENV_WRAPPER` | How secrets are injected: `doppler run --project X --config Y -- …`, script already wraps Doppler, or plain env |
| `HISTORY_STORE` | e.g. Drizzle `__drizzle_migrations` / schema pin notes from README |
| `FORBIDDEN` | Commands docs disable (e.g. kectil `db:push` / `db:push:force`) |
| `SEED_POLICY` | Content/seed scripts are **never** part of `/prb` migrate unless the user explicitly requests a named seed against a confirmed env |

### Common stack patterns (examples only — override with repo docs)

| Pattern | Typical apply |
|---------|----------------|
| Drizzle + `db:migrate` + Doppler | `doppler run --project <p> --config <production> -- <pkg> run db:migrate` (or repo script that already includes `doppler run`) |
| Drizzle monorepo filter | `doppler run … -- pnpm --filter <pkg> db:migrate` |
| SQL files + `psql` (no drizzle-kit ledger) | Doppler + `psql "$DATABASE_URL_UNPOOLED" -X --set ON_ERROR_STOP=1 --single-transaction -f <file>` **only** when docs say so |
| Prisma | Repo’s `prisma migrate deploy` (production) — not `migrate dev` |

### Hard discovery rules

- **Never** invent Doppler project/config names. If unknown: `doppler configs` / read docs; if still unclear, **stop and ask**.
- **Never** print `DATABASE_URL`, Doppler secret values, or connection strings. Redact hosts if needed (`doppler run -- …` with a check that `DATABASE_URL` is set without printing it).
- **Never** run `db:push` / `drizzle-kit push` / `prisma db push` against production unless the user explicitly orders it **and** docs allow it (rare). Prefer versioned migrate.
- Prefer **unpooled** URLs when the project documents `DATABASE_URL_UNPOOLED` for migrations (Neon); use whatever the project’s migrate path already selects.
- Do not run repair/history-rewrite scripts (`db:repair:*`) unless docs say the target DB needs them **and** the user confirms.

---

## 3. Classify migration risk

Skim new/changed SQL (and custom migrate scripts) in the ship set:

| Class | Signals | `/prb` behavior |
|-------|---------|-----------------|
| **Expand / additive** | `CREATE TABLE`, `ADD COLUMN`, new indexes, `IF NOT EXISTS`, non-destructive `CREATE TYPE`, data backfills that only insert/update nullable or new columns | Auto-run production migrate at the **pre-merge** gate when quiet window allows merge |
| **Expand + data** | Backfills, `UPDATE`/`INSERT` on existing rows, CMS content migrations | Auto-run only if SQL looks idempotent **or** docs mark it safe; otherwise pause for user confirmation |
| **Contract / destructive** | `DROP TABLE/COLUMN`, `RENAME`, non-null without default, type changes that rewrite data, truncates | **Do not auto-run.** Stop, summarize SQL, require explicit user approval and ordering |
| **Unknown / mixed** | Large custom SQL, unclear scripts | Treat as destructive: stop and ask |

Also respect expand/contract discipline:

1. **Expand** (additive) must land before code that **requires** new columns/tables.
2. **Contract** (drops) must land only after code no longer reads/writes removed objects (often a **later** ship).

If this ship both requires new columns **and** drops old ones in one migration, **stop** and ask — that is a high-risk single cutover.

---

## 4. When to run migrations during `/prb`

### Default production timing (when project docs do not specify otherwise)

| Step | When | Action |
|------|------|--------|
| **Inventory** | Phase 0 / after push | Detect `MIGRATIONS_IN_SHIP`, discover procedure |
| **Preview (optional)** | Early babysit if preview/stg Doppler exists and preview apps share a DB that needs schema | Apply **preview/stg** migrate with the same discovered command + preview config so preview CI/UI is not broken. Skip if no preview DB or docs say not to |
| **Production (required if ship has migrations)** | **After** merge criteria pass, **before** `gh pr merge` | Run **production** migrate successfully |
| **Verify** | Immediately after migrate | Confirm command exit 0; re-read project notes for any post-migrate check (e.g. schema audit script) |
| **Merge** | Only after production migrate OK (additive path) | `gh pr merge` so production deploy picks up code against ready schema |
| **Post-merge** | After merge + local main sync | If migrate was blocked/skipped earlier by mistake, **do not leave prod code ahead of schema** — run production migrate immediately and report |

**Why migrate before merge (default):** New code on `main` often deploys within minutes of merge. Missing columns/tables break production. Additive schema is almost always backward-compatible with the previously deployed app, so applying SQL first is the safer default.

**Override when project docs say otherwise:** Follow the repo (e.g. “migrate immediately after shipping”). Still never leave production app code requiring schema that is not applied.

**`--no-merge`:** Still discover and report pending production migrations. **Do not** apply production migrate unless the user explicitly asks to migrate production without merging (unusual). Call out: “PR not merged; production migrate not run.”

**`--skip-migrations`:** Skip apply gates; report loud warning that production schema may lag code. Do not use this by default.

### Destructive path

1. Do not merge automatically.
2. Present file list + risk summary.
3. Wait for explicit user instruction (order of migrate vs merge, approval).
4. Only then run the project’s production migrate command.

### Fix-push during babysit

If a mid-window fix adds new migrations on `dev`:

1. Re-run inventory.
2. Re-apply **preview** migrate if you use preview.
3. Production migrate still runs at the pre-merge gate with the full final set (migrate tools are incremental / journaled).

---

## 5. Execution checklist (production)

1. `git fetch origin` and confirm `dev` (or PR head) SHA that will merge includes the migration files you will apply.
2. Working tree clean of unrelated dirt; on a branch that has those commits (usually local `dev` after push).
3. Doppler (or env) authenticated for the project; do not print secrets.
4. Build the command from discovery — examples of **shape** only:

```bash
# Drizzle via package script + explicit production Doppler (when script does not pin config)
doppler run --project "$DOPPLER_PROJECT" --config "$DOPPLER_PRODUCTION" -- $PKG run db:migrate

# When package.json already wraps doppler run for the active setup config:
# switch shell Doppler config or pass explicit project/config per docs, then:
$PKG run db:migrate
```

5. Run with a generous timeout; migrations can be slow on Neon.
6. On **non-zero exit**: **do not merge**. Capture error text (redact secrets). Diagnose (wrong config, history mismatch, destructive SQL). Offer repair only if project documents it.
7. On **success**: note time + SHA + config name (not secrets) in the session log, then proceed to merge.
8. Never chain content seeders, `db:push`, or destructive repair after a successful migrate as part of `/prb`.

### Redacted readiness check (optional, before migrate)

```bash
doppler run --project "$DOPPLER_PROJECT" --config "$DOPPLER_PRODUCTION" -- \
  sh -c 'test -n "$DATABASE_URL" && echo "DATABASE_URL is set (value redacted)"'
```

Do not `echo "$DATABASE_URL"`.

---

## 6. Failure modes

| Failure | Action |
|---------|--------|
| No migrate script / no docs | Stop; ask user how this repo applies production schema |
| Doppler auth missing | Stop; ask user to auth; do not fall back to stale `.env` production URLs unless user explicitly provides a safe method |
| Migrate fails | Block merge; report; do not retry blindly more than once without reading the error |
| History/journal mismatch | Point at project repair docs; do not rewrite journals casually |
| User veto | Skip migrate and merge |
| Migrations only on wrong branch | Get them onto `dev` first |

---

## 7. Report fields (append to Phase 5)

```markdown
**Migrations:** none in ship | applied production (`<config>`) @ <time> | blocked (<reason>) | skipped (--skip-migrations|--no-merge)
**Migrate command shape:** <pkg> / drizzle|prisma|psql|custom (no secrets)
**Risk class:** additive | data | destructive | n/a
```
