# Coverage

The inventory JSON is the universe. Every `files[].path` gets a row.

## Assignment

```text
path → <Canonical> | not-a-feature
reason: one line
```

Write `$SCRATCH/coverage.md` (or `.jsonl`) before slicing features.

**Feature** — this path implements or substantially configures a capability
(see `SKILL.md` granularity) **and** that capability has a real surface.
One path may serve several features; list the primary and note others in
`reason`.

Do **not** assign:

- `app/api/**/route.ts` → **Public API** unless it is a documented/public/partner API
- a file that merely contains the string `webhook` / `upload` / `session` → that canonical
- `app/admin/**` → **Admin** unless there is a distinct operator console (otherwise the capability those pages implement)
- leftover buckets **Application**, or **Gameplay** on a non-game

**not-a-feature** — infra, generated, tests-only harness, one-off util,
tooling, lockfile-adjacent config, secrets path, CI, design tokens with no
product behavior, vendored-adjacent leftovers the script kept.

Those paths still feed the project Index **Stack** table when they name a
vendor (`vercel.json`, env examples, ORM config). Do not assign the vendor
as a canonical. Stack shape: [`templates.md`](templates.md).

Tests that **are** the spec for a product capability (e.g. impersonation e2e)
assign to that feature, not `not-a-feature`.

README/PRD/VISION: use as hints for names and flows; still assign the path
(`not-a-feature` if it is only docs, unless it *is* the product: a docs site).

## Gate

A run is incomplete while any inventory path lacks a row. Do not write notes
until the gate passes.

Zero features with full assignment is allowed (tiny repo); say so in handoff.

## Hits

`hits[]` and `hit_counts` are searchlights. If `impersonat` has hits, you
must either create/attach **Impersonation** or say in coverage why those
hits are not that feature (string in a comment, dependency name, etc.).

## Workers (optional)

Use extra agents only when `file_count` > 400. Split by top-level directory
(or obvious packages in a monorepo). Each worker:

- Gets a slice of `files[]` plus the granularity rule and starter aliases
- Returns coverage rows only
- Does not write the vault, config, or catalog
- Does not confirm names with the user

Merge rows here. Then this run does catalog confirmation and vault writes.

Worker prompt (fill the slice):

```text
You classify paths for /map. You do not write files.

Feature = a capability you could ship, drop, or copy on its own.
Auth ≠ Impersonation. A helper or button is not a feature.
A vendor is not a feature (`vercel.json` → not-a-feature).

Assign every path in the JSON slice to a Canonical name or not-a-feature
with a one-line reason. Return a markdown table: path | assignment | reason.
No extra prose.
```

## Inventory JSON (from `scripts/inventory`)

```json
{
  "repo_root": "/abs/path",
  "git": true,
  "file_count": 412,
  "hit_counts": {"impersonat": 6},
  "files": [
    {
      "path": "src/admin/impersonate.ts",
      "language": "ts",
      "bytes": 1234,
      "skipped_read": false,
      "hits": ["impersonat"]
    }
  ],
  "skipped": {
    "dir": 0,
    "ignored": 0,
    "binary": 0,
    "lockfile": 0,
    "oversized": 0,
    "secret_env": 0
  }
}
```

`skipped_read: true` (env files, binaries, oversized): still assign the path
(usually `not-a-feature`). Never open the file for contents.
