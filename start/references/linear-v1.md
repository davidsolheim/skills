# V1 queue for `/start`

Used by `/start` Phase 4. Write leaves into `.WCP/issues/open/` per [`../../docs/wcp-queue.md`](../../docs/wcp-queue.md). Do not create a Linear project. Do not call Linear. Ticket quality is **not** forked: use

- [`../../issue/references/execution-ready-bar.md`](../../issue/references/execution-ready-bar.md)
- [`../../issue/references/issue-body-template.md`](../../issue/references/issue-body-template.md)
- [`../../issues/references/decomposition.md`](../../issues/references/decomposition.md)
- [`../../issues/references/epic-body-template.md`](../../issues/references/epic-body-template.md)
- [`../../issue/references/direction-conflict.md`](../../issue/references/direction-conflict.md)

Do not call Linear.

## Database

Follow [`../../docs/notion-issues.md`](../../docs/notion-issues.md). Title is the lowercase repo slug. Description and every row's Repo property are the normalized origin URL. Reuse that database when it already exists.

## Queue

Write each V1 leaf as a file in DEST `.WCP/issues/` ([`../../docs/wcp-queue.md`](../../docs/wcp-queue.md)). Create `open/`, `in-progress/`, `in-review/`, `done/`, `canceled/`, and `blocked/` when they are missing. Ids are the next number under `.WCP/issues/`. There is no epic file.

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

## Leaves

There is no epic file. Put `batch: start-v1-<slug>` in the leaf body when a batch name helps.

Each create leaf: full `/issue` template (include `temp_id` and `class`).

Create gate (fail closed) — same as `/issue` 5A. Paths must exist in DEST now
(starter paths count).

Filing:

- `status: open`, empty `assignee` and `lease_expires`, file in `open/`
- Notion Status `open` after the file write
- `priority`: identity and blocking schema are high; core features are high or normal; polish is normal or low
- A hard dependency is a file in `blocked/` with `reason: blocked by <id>` and Notion Status `blocked`

User report on each leaf: quote the vision V1 bullet.

Platform / stack on every leaf:

- Canonical: Next.js App Router, Better Auth, Neon/Drizzle migrations, Doppler, Vercel
- Abandoned: `db:push`, Server Actions for mutations, BlockNote, ecommerce (unless V1 exception)

## Board snapshot

Read `.WCP/issues/` before writing so a second `/start` does not file the same leaf again.

## Draft mode

`--draft`: print the leaf bodies in chat. Do not write the files or Notion rows.

## Notion failure

Say which upsert failed. Keep DEST docs and any files already written. The files are still the queue. Phase 6 still runs unless the user passed `--no-build`, `--docs-only`, or `--draft`.
