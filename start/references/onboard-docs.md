# Onboard AGENTS.md, VISION.md, README

Used by `/start` Phase 3. All paths are in DEST.

**Authority:** DEST `AGENTS.md` **First-run onboard** (copied from the
template). Questions, prefill rules, and the onboarded AGENTS.md shape live
there. This file is a backup of file shapes if you need them after the
questionnaire is gone.

Do **not** patch the first-run file in place and leave the marker. Overwrite
`AGENTS.md` in full with the onboarded shape.

## VISION.md (create)

This file is the V1 scope lock. Nested `/solve` and the Notion queue obey it.
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

## Notion

- Project page: <url or pending>
- Issues database: `<slug>` — description is the origin URL; pending until that database exists
- Do not create Linear issues. Do not write `.linear-project`.

## Success

- <how we know V1 is real: routes a user can complete>
```

**V1 list rules:**

- 3–8 must-ship bullets. If the brief is huge, cut to a vertical slice and put
  the rest under Later.
- Each bullet is an outcome (`shops file a claim and verify email`), not a
  stack task (`add Postgres`).
- Always imply product identity (name, metadata, home, footer) if not listed.

If the Notion issues database is not created this run, Project page and
Issues database may say `pending`.

## AGENTS.md (overwrite)

Use the **Onboarded AGENTS.md** block from the first-run file (filled).
Must not contain `first-run: starter-onboard`. Must include Notion, Product,
Secrets (`<slug>` / `development`), Database, Auth, Conventions, and:

```markdown
## wcp

This checkout may have many coding agents on one local `dev` working tree. The issue files are the protocol. There is no daemon.

- Skill: `water-cooler-protocol`
- `in-progress/` means an agent is editing that issue on this machine. Read those files and work around the paths they list.
- Move your issue to `done/` when the writing is finished. Do not commit while any issue is in `in-progress/`.
- When `in-progress/` is empty and you have no further task, commit, push `dev` to `origin/dev`, and move those `done/` issues to `deployed-dev/`.
- `deployed-main/` is only for a commit that is on `origin/main`.
- Commit `.wcp/issues/` and `.wcp/tracker.md`. Do not commit any other path under `.wcp/`. Do not reset, checkout, or stash another agent's work.
- External tracking is optional. `.wcp/tracker.md` is `none` until the user names `notion`, `linear`, `jira`, or another tool. A named tracker gets the same updates.
```

## README.md (rewrite)

Not a clone of the template README. Shape like a product README
(thesingularityreport-com / cblacklist-com):

1. `# <Product name>`
2. One-paragraph job + URL if known
3. Credit: bootstrapped from [next-starter-template](https://github.com/teton-web/next-starter-template) (**MIT** © Teton Web Ventures LLC), content-only / no starter git history
4. Stack bullets (current starter facts: Next 16, Drizzle/Neon, Better Auth, Resend, Tailwind 4, shadcn, CMS, media, site gate)
5. Getting started: **this** slug (`git clone` this repo, `doppler setup --project <slug>`)
6. Auth / structure / scripts — keep useful starter sections, retarget names
7. PRs target `origin/dev`; do not open PRs against `main`

Installation commands must not say `cd next-starter-template`.

## Do not write `.linear-project`

Do not create that file. Do not call Linear. If a leftover `.linear-project`
exists, leave it on disk and do not treat it as the tracker.

## Commit

Stage only onboard + identity files. Commit on `main`:

```text
docs: onboard <product name>
```

Merge `main` into `dev` (create `dev` from `main` if needed).
