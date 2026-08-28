# Linear capture for `/start` V1

Used by `/start` Phase 4. Ticket quality is **not** forked: use

- [`../../issue/SKILL.md`](../../issue/SKILL.md) Phase 4 quality bar (and `references/execution-ready-bar.md` if present)
- [`../../issue/references/issue-body-template.md`](../../issue/references/issue-body-template.md)
- [`../../issues/references/decomposition.md`](../../issues/references/decomposition.md)
- [`../../issues/references/epic-body-template.md`](../../issues/references/epic-body-template.md)
- `../../issue/references/direction-conflict.md` if present

`search_tool` → `use_tool`. Literal markdown newlines.

## Team

Confirmed in Phase 1. If more than one Linear MCP server is connected, use the
one whose `list_teams` includes the resolved team.

`list_teams` before create. Do not invent a team.

## Project

**Do not** file a new product on an existing **umbrella / platform** board that
belongs to another product. Create (or reuse) a Linear project whose name
**is this product**.

**Existing `/start`:** if `.linear-project` or AGENTS already names a project
and `list_projects` finds it on the team → **reuse**. Do not create a second
project with a similar name.

1. `list_projects` `team` + `query` = product name / slug / `.linear-project`
2. If an active project already **is** this product (name/slug match) → reuse it
3. Else `save_project` without `id`:

```text
name: <Product name>          # human name, e.g. Acme App
setTeams: ["<team>"]
summary: <job in ≤255 chars>
description: <markdown: intent, V1 bullets, repo path, VISION.md pointer>
state: started                # if the API accepts a started-type; else omit
links: [{ url: <github or omit>, title: Repo }]
```

4. Capture name, id, url, team key/prefix
5. Patch DEST `AGENTS.md`, `VISION.md` Linear section, `.linear-project`

## What to ticket (V1 only)

Source dump = `VISION.md` **V1 Must ship** (+ implied identity if missing).

Investigate DEST (starter files are the “current behavior”).

### Do not file

- “Add Next.js / Tailwind / shadcn”
- “Add Better Auth / Neon / Drizzle / Doppler / Resend” as greenfield work
- “Add CMS / media library / contact / privacy / terms / login / admin” unless
  V1 **changes** their behavior or copy/IA in a shippable way
- Later-section bullets
- Production DNS, custom domain cutover, or “deploy to Vercel” unless the user
  asked for those in V1 (even then: no production DNS without explicit ask)

### Do file

| Class | Typical V1 leaves |
|-------|-------------------|
| `foundation` | Product identity (metadata, title, footer, `lib/seo`, home frame); schema/migrations **this product adds** |
| `feature` | Each distinct V1 user outcome (one leaf per outcome) |
| `content` | Privacy/terms/home copy **specific to this product** if not folded into identity |
| `polish` | Only if V1 success requires it |

Identity is usually **one** foundation leaf, not six micro-tickets.

Prefer 4–12 leaves. Split if a cheap model would need a multi-day plan.

## Epic

Default: one epic `V1 – <Product>`. Body from epic-body-template; source
`/start`; **do not implement the shell**.

If that epic already exists on this project (existing `/start` retry), **reuse
it**. File only V1 leaves that are not already open/done (duplicate scan).

`--no-epic` is not a `/start` flag; only skip the epic if exactly one leaf.

## Leaves

Each create leaf: full `/issue` template (include Batch metadata:
`temp_id`, `class`, `batch: start-v1-<slug>`).

Create gate (fail closed) — same as `/issue` 5A. Paths must exist in DEST now
(starter paths count).

Filing:

- Unassigned, Backlog/Todo (or team default unstarted)
- **Not** In Progress, **not** Done, **not** claimed
- `parentId` = epic
- `project` = this new project
- `priority`: identity + blocking schema = High (2); core features = High or Medium; polish = Medium/Low
- `blockedBy` after ids exist
- Labels only if they already exist on the team

User report on each leaf: quote the vision V1 bullet.

Platform / stack on every leaf:

- Canonical: Next.js App Router, Better Auth, Neon/Drizzle migrations, Doppler, Vercel
- Abandoned: `db:push`, Server Actions for mutations, BlockNote, ecommerce (unless V1 exception)

## Board snapshot

New project → usually empty. Still `list_issues` once on this project
(backlog/unstarted/started) so we do not duplicate a retry of `/start`.

Do not snapshot some other product’s board looking for “similar” work to attach to.

## Draft mode

`--draft`: no `save_project` / `save_issue`. Output the project markdown +
every leaf body in chat.

## Linear failure

Say which call failed. Keep DEST docs. Print project + leaf drafts. Phase 6
build does **not** run unless the user already passed `--no-linear` and wants
vision-driven implement.
