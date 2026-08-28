# Onboard AGENTS.md, VISION.md, README

Used by `/start` Phase 3. All paths are in DEST.

**Authority:** DEST `AGENTS.md` **First-run onboard** (copied from the
template). Questions, prefill rules, and the onboarded AGENTS.md shape live
there. This file is a backup of file shapes if you need them after the
questionnaire is gone.

Do **not** patch the first-run file in place and leave the marker. Overwrite
`AGENTS.md` in full with the onboarded shape.

## VISION.md (create)

This file is the V1 scope lock. Nested `/solve` and Linear leaves obey it.
Filename is **`VISION.md`** (not `vision.md`). On existing repos, rename
lowercase `vision.md` if that is the only copy.

```markdown
# <Product name>

## Intent

<one paragraph: who it is for, what job it does, why it exists now>

## Users

- Primary: <role + job-to-be-done>
- Secondary: <or none>

## V1 (this /start run)

Must ship:

- <user-visible outcome 1>
- <outcome 2>
- …

Must not ship (later / out of scope for this run):

- <explicit cut>

## Later

- <nice-to-haves that must not leak into V1 tickets>

## Non-goals

- Ecommerce, i18n URL prefixes, BlockNote, BotID, catalogs — unless the brief
  explicitly demands one (then call that out as a starter-boundary exception)
- Rebuilding Better Auth / Neon / CMS / media / Doppler from scratch

## Brand / voice

- <short; infer from brief; “match a calm professional marketing site” is enough>

## Stack (locked)

Next.js 16 App Router, Better Auth, Neon + Drizzle **migrations only**, Doppler,
Resend, Tailwind 4, shadcn/ui, starter CMS + media library. Mutations = Route
Handlers + Zod, not Server Actions.

## Linear

- Team: <name> (`PREFIX`)
- Project: <name>   <!-- fill URL after Phase 4 -->
- Identifiers: `<PREFIX>-*`

## Success

- <how we know V1 is real: routes a user can complete>
```

**V1 list rules:**

- 3–8 must-ship bullets. If the brief is huge, cut to a vertical slice and put
  the rest under Later.
- Each bullet is an outcome (`users list an item and confirm pickup`), not a
  stack task (`add Postgres`).
- Always imply product identity (name, metadata, home, footer) if not listed.

If Linear is not created this run, still fill Team from Phase 1; Project may
say `pending`.

## AGENTS.md (overwrite)

Use the **Onboarded AGENTS.md** block from the first-run file (filled).
Must not contain `first-run: starter-onboard`. Must include Linear, Product,
Secrets (`<slug>` / `development`), Database, Auth, Conventions.

## README.md (rewrite)

Not a clone of the template README. Shape like a product README:

1. `# <Product name>`
2. One-paragraph job + URL if known
3. Credit: bootstrapped from [next-starter-template](https://github.com/davidsolheim/next-starter-template) (**MIT** © David Solheim), content-only / no starter git history
4. Stack bullets (current starter facts: Next 16, Drizzle/Neon, Better Auth, Resend, Tailwind 4, shadcn, CMS, media, site gate)
5. Getting started: **this** slug (`git clone` this repo, `doppler setup --project <slug>`)
6. Auth / structure / scripts — keep useful starter sections, retarget names
7. PRs target `origin/dev`; do not open PRs against `main`

Installation commands must not say `cd next-starter-template`.

## .linear-project

Single line, no extra markup:

```text
<Linear project name>
```

Same string used in `save_project` `name` (or the name you will use in Phase 4).

## Commit

Stage only onboard + identity files. Commit on `main`:

```text
docs: onboard <product name>
```

Merge `main` into `dev` (create `dev` from `main` if needed).
