# Note templates

Fill every generated section. Mermaid is required on project notes (sequence
when there is a flow; flowchart otherwise). No screenshots, no canvas files.

## Wikilinks

| From | To | Link |
|------|----|------|
| Project note | Sibling feature | `[[Canonical]]` |
| Project note | Pattern | `[[patterns/Canonical]]` |
| Pattern / home / catalog | Project feature | `[[projects/<slug>/Canonical]]` |
| Anywhere | Pattern | `[[patterns/Canonical]]` |

Filenames = canonical name + `.md` (spaces allowed). Title = canonical name.

## Frontmatter

### Project feature

```yaml
---
type: project-feature
canonical: Impersonation
project: ore-max
status: active
aliases: []
depends_on:
  - Auth
related: []
sources:
  - src/admin/impersonate.ts
repo_path: /abs/path/to/repo
updated: YYYY-MM-DD
---
```

`status`: `active` | `stale`. `sources`: key files only, not every coverage
path. `depends_on` / `related`: canonical names.

### Pattern

```yaml
---
type: pattern
canonical: Impersonation
status: active
aliases: [login-as, act-as]
projects: [ore-max]
updated: YYYY-MM-DD
---
```

### Project index

```yaml
---
type: project-index
slug: ore-max
repo_path: /abs/path/to/repo
updated: YYYY-MM-DD
---
```

## Project note body (order is required)

````markdown
# Impersonation

## What it is

3–8 sentences. What the user or operator can do. Not a file list.

## How it works

User/operator flow in prose.

## Sequence

```mermaid
sequenceDiagram
  actor User
  participant App
  participant Auth
  User->>App: request
  App->>Auth: check
```

## Connections

- **Depends on:** [[Auth]]
- **Unlocks:**
- **Related:** [[patterns/Impersonation]]

## Surfaces

Routes, UI entry points, APIs.

## Implementation map

| Path | Role |
|------|------|
| src/admin/impersonate.ts | Starts impersonation session |

## Data and config

Models, tables, env keys (**names only**, never values).

## Gotchas

Leapfrog gold: edge cases, coupling, “don’t copy this”.

## Port to a new app

1. **Prerequisites** — other features that must exist first
2. **Stack assumptions** — framework, auth, db this recipe expects
3. **Recreate** — files/concepts to build, in order
4. **Env** — variable *names*
5. **Verify** — how you know it works
6. **Do not copy** — repo-specific hacks

## Human notes
````

Empty Human notes: leave the heading and a blank line.

## Pattern note body

```markdown
# Impersonation

Portable definition (what the capability *is*, not how one repo did it).

## Across projects

| Project | How they do it | Worth stealing | Caveats | Note |
|---------|----------------|----------------|---------|------|
| ore-max | … | … | … | [[projects/ore-max/Impersonation]] |

## Default recipe

Best of what is in the vault when starting fresh. Point at the project note
to copy from.

## Aliases

From the catalog.

## Human notes
```

First sighting: one table row; default recipe can match that project’s port
section. Later runs: add/update the row; rewrite default recipe if a better
implementation appeared.

## Indexes

**`projects/<slug>/Index.md`** — list active features, then stale.

**`patterns/Index.md`** — all canonical patterns.

**Vault `Index.md`** — link each project index + pattern index + catalog.
Keep the intro line; replace the “no projects” placeholder once any exist.

## Re-run

1. Read the existing note (if any).
2. Capture from `## Human notes` through EOF (exact heading). If missing,
   captured body is empty.
3. **`status: active` and still present:** rewrite frontmatter + generated
   sections 1–9 from current code; append `## Human notes` + captured body.
4. **Gone from this inventory:** set `status: stale`, bump `updated`, add a
   one-line stale callout if not already present, **keep** the previous
   generated sections (do not invent a new flow from empty evidence); keep
   Human notes.
5. Do not delete the file. Do not merge Human notes into generated sections.

Anything the user wrote *above* `## Human notes` is generated and will be
replaced. Point that out in the first-run handoff once.
