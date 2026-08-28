# Existing repo — validate bones, then repair

Used by `/start` when `MODE=existing`. Do **not** rsync the template onto
DEST. Do **not** wipe product code.

Read this file in Phase 2 (existing). Repair in Phase 3.

## Hard fail vs repair

| Class | Meaning | Action |
|-------|---------|--------|
| **Bone** | Starter framework/layout that V1 assumes | Missing → **stop**. Do not convert a different stack. |
| **Doc / identity** | AGENTS, VISION, README, Linear binding, Doppler slug | Wrong or missing → **repair** (onboard). |
| **Git** | `main` + lowercase `dev` | Missing `dev` → create from `main`. Do not invent a second remote. |

Report every check as `pass` / `repair` / `fail` before writing.

---

## Bones (hard)

DEST must look like a next-starter-template descendant. Check by **reading
files**, not guessing from the README.

| Check | Pass when |
|-------|-----------|
| App Router | `app/` exists with `app/layout.tsx` |
| Next | `package.json` depends on `next`; `next.config.*` present |
| Auth | `lib/auth.ts` or Better Auth setup; `app/(auth)/login/` ; `app/admin/` |
| Login path | `/login` not `/admin/login` (`proxy.ts` or auth routes) |
| DB | `drizzle.config.ts`, `drizzle/`, `lib/db/schema/` |
| Migrations only | `package.json` `db:generate` / `db:migrate` exist; `db:push` **disabled** (script exits 1) |
| Doppler | `doppler.yaml`; `dev` / `db:*` scripts use `doppler run` |
| UI kit | Tailwind + `components.json` / `components/ui/` |
| Mutations | Route Handlers under `app/api/` (spot-check; do not require zero Server Actions in forms) |
| Package manager | `bun.lock` (or bun.lockb) |
| Verify scripts | `lint`, `typecheck`, `test` in `package.json` |

**Fail the run** (no scaffold overlay) if any of:

- No `app/` App Router
- No Next.js
- Shopify Hydrogen / other storefront as the primary app with no starter `lib/db`
- Convex / Prisma-only / no Drizzle when the product claims this starter
- `package.json` name is `next-starter-template` **and** origin is the public template (wrong tree — that is template maintenance, not `/start`)

On hard fail: table of failed bones, what stack this actually is, and stop.
Ask the user to `/start` a **new** DEST or confirm a different tool.

Auth.js/NextAuth instead of Better Auth: **repair-note**, not hard fail, if the
rest of the bones match (older fork). Do not silently migrate auth in validate;
file or repair only if Phase 6/V1 needs it and VISION says so.

---

## Docs and identity (repair)

Infer slug from `package.json` `name`, Doppler project, or directory name.
Infer product name from README `#` heading or VISION title.

| Check | Pass when | Repair |
|-------|-----------|--------|
| `AGENTS.md` exists | file present | Restore onboarded shape from template AGENTS **Onboarded** block (need answers) |
| No first-run marker | no `<!-- first-run: starter-onboard -->` | Run first-run onboard (questions + overwrite) |
| AGENTS Linear | `## Linear` with team + project | Add from answers / `.linear-project` |
| AGENTS Secrets | Doppler project is **this** slug, not `next-starter-template` | Retarget |
| AGENTS Database / Auth / Conventions | migrations-only, `/login`, no `db:push`, `dev` branch | Fill from platform section |
| `VISION.md` (not `vision.md`) | exists; **Intent** + **V1 Must ship** (3–8 outcomes) | Create/upgrade via onboard-docs; rename lowercase `vision.md` → `VISION.md` |
| `README.md` | product title + job; install uses **this** slug; no `cd next-starter-template` as *this* repo | Rewrite product README |
| `.linear-project` | one line, product Linear name | Write it |
| `package.json` `name` | equals slug, not `next-starter-template` | Retarget |
| `doppler.yaml` `setup.project` | equals slug, not `next-starter-template` | Retarget |
| `doppler:setup` script | `--project <slug>` | Retarget |
| Starter leftover copy | home/footer/metadata may still say starter — product identity can be a V1 leaf | Do not hard-fail; ticket in Phase 4 if still generic |

If both `VISION.md` and `vision.md` exist, keep **`VISION.md`**, merge unique
V1 bullets, delete or stop tracking the lowercase file.

First-run marker present → full Phase 3 onboard (prefill from existing
README/VISION/package.json; ask only gaps).

Docs already onboarded and checks pass → **no rewrite**. Still list them as
pass in the handoff.

---

## Git (repair)

| Check | Repair |
|-------|--------|
| Git repo | If not a repo: `git init` + `main` only if the user intended `/start` here; else stop |
| `main` | Do not rewrite history |
| lowercase `dev` | Create from `main` if missing; never create `Dev` |
| `origin` is the **template** | Stop and tell the user; do not push product commits to the starter |

Do not force-push. Do not delete branches.

---

## Linear (existing)

If `.linear-project` / AGENTS names a project that `list_projects` finds on
the team → **reuse**. Do not `save_project` a duplicate.

If a `V1 – <Product>` epic already exists on that project → reuse; only file
**missing** V1 leaves (duplicate scan). Do not file a second epic.

---

## Output before Phase 3

```markdown
**Mode:** existing
**DEST:** <path>
**Bones:** pass N · fail F
**Docs/identity:** pass N · repair R
**Git:** pass | repair `dev`
```

If `fail F > 0` on bones → stop (no Phase 3–6).
If only repairs → continue Phase 3 with the repair list as the write set.
