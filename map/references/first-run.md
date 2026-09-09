# First run and config

Machine config (not in the vault): `${GROK_HOME:-$HOME/.grok}/map.json`

```json
{
  "vault_path": "/absolute/path/to/Maps",
  "project_roots": [],
  "created_at": "2026-09-09T00:00:00Z",
  "updated_at": "2026-09-09T00:00:00Z"
}
```

No secrets. Safe to delete → wizard runs again.

If `map.json` is missing and `${GROK_HOME:-$HOME/.grok}/feature-map.json`
exists, copy it to `map.json` (do not delete the old file). Prefer
`map.json` when both exist.

## When to run the wizard

Run it if any of these is true:

- Config file missing
- `vault_path` missing or empty
- `vault_path` is not a directory (gone, never created, or a file)
- User is running `all` and `project_roots` is empty

Do not re-ask vault location on later runs when the path still exists.

## Wizard

Ask **in this order**. Wait for answers before creating files.

### 1. Vault location

> Where should the Maps Obsidian vault live?
> Suggested: `~/Documents/Maps`
> (A new folder. You will add it as a vault in Obsidian once.)

Expand `~`. Resolve to an absolute path. If they pick an existing folder that
already has `catalog.md` and `Index.md`, **use it as-is** (do not overwrite).
If the folder is missing or empty, create and seed.

### 2. Project roots (optional)

> Optional: absolute paths of repos for `/map all` (one per line, or skip).
> You can add these later by editing the config.

Skip is allowed. `all` with an empty list → ask this question then, do not
scan the home directory.

### 3. Write config

Set `created_at` on first write; always bump `updated_at`.

## Seed (new or empty vault only)

Create:

```text
<vault>/.obsidian/app.json
<vault>/.obsidian/graph.json   # color groups: patterns vs projects
<vault>/Index.md
<vault>/catalog.md
<vault>/projects/.gitkeep
<vault>/patterns/Index.md
```

Home Index, catalog list, and graph groups: [`obsidian.md`](obsidian.md).

### `Index.md`

```markdown
---
type: vault-index
updated: YYYY-MM-DD
---

# Maps

Cross-project map of real codebases: features, stack, and leapfrog patterns.

## Start here

_No patterns yet._

## Projects

_No projects mapped yet._

## Patterns

See [[patterns/Index]] and [[catalog]].
```

### `catalog.md`

Copy the **starter table** from [`catalog.md`](catalog.md) (Canonical + Aliases
columns; Pattern and Projects empty). Do not invent extra rows.

### `patterns/Index.md`

```markdown
---
type: pattern-index
updated: YYYY-MM-DD
---

# Patterns

Shared feature pages. Each pattern links at least one project note.

_No patterns yet._
```

`.obsidian/app.json`:

```json
{}
```

`.obsidian/graph.json` `colorGroups`: `path:patterns` (warm), `path:projects` (cool), `tag:#status/stale` and `tag:#status/empty` (muted). Exact RGB is in [`obsidian.md`](obsidian.md) — seed if the file is missing or `colorGroups` is empty.

Tell the user once: add `<vault_path>` as a vault in Obsidian (Open folder as
vault). Do not try to launch Obsidian.

## Config updates later

- Changing vault path: user says so, or path is gone → ask again, update
  `vault_path`. Do not move files unless they ask.
- Adding roots: merge into `project_roots` (unique absolute paths).
