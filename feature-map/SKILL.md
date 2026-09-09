---
name: feature-map
description: >
  Use when the user runs /feature-map, /feature-map all, or /feature-map <path>,
  or asks to map a project's features into Obsidian, extract features from a
  codebase, build a cross-project feature catalog, or leapfrog implementations
  from past apps.
argument-hint: "[all | path]"
metadata:
  short-description: "Map project features into an Obsidian vault"
---

# /feature-map — Codebase → Obsidian feature vault

Extract **features** from a codebase into a shared Obsidian vault so later
projects can leapfrog. One note per feature per project, plus a shared pattern
note for the same capability across apps.

This skill is **read-only on the app repo**. It writes the Feature Maps vault
and `${GROK_HOME:-~/.grok}/feature-map.json` only.

## Operating contract

1. **Feature** = a capability you could ship, drop, or copy on its own. Auth
   and Impersonation are both features; Impersonation `depends_on` Auth. A
   login button or `validateSession` helper is not a feature. Prefer split
   over stuff.
2. **Inventory script is the universe.** Every kept path is assigned a feature
   or `not-a-feature`. No silent skips. Do not eyeball the tree instead of
   running the script.
3. **Catalog names.** Match aliases in the vault `catalog.md`. Unknowns →
   **one** confirmation list, then write. Do not guess a second canonical for
   the same capability.
4. **Re-run.** Refresh generated sections. Never overwrite `## Human notes`.
   Missing features become `status: stale` (keep the file). Do not delete notes.
5. **Pattern notes.** First sighting still creates `patterns/<Canonical>.md`.
   Later projects add a comparison row and update the default recipe if better.
6. **`all` is sequential.** Full pipeline per root, then the next. Catalog
   updates after each project so the next one can match.
7. **No browser, no screenshots, no Linear, no app commits, no other vaults.**

## Invocation

```text
/feature-map
/feature-map /path/to/repo
/feature-map all
```

| Arg | Target |
|-----|--------|
| *(none)* | Current workspace (must be a project root) |
| path | That repo |
| `all` | Each `project_roots` entry, one at a time |

## Skill paths

```text
FEATURE_MAP_SKILL_DIR = directory containing this SKILL.md
INVENTORY             = $FEATURE_MAP_SKILL_DIR/scripts/inventory
FIRST_RUN_MD          = $FEATURE_MAP_SKILL_DIR/references/first-run.md
TEMPLATES_MD          = $FEATURE_MAP_SKILL_DIR/references/templates.md
CATALOG_MD            = $FEATURE_MAP_SKILL_DIR/references/catalog.md
COVERAGE_MD           = $FEATURE_MAP_SKILL_DIR/references/coverage.md
CONFIG                = ${GROK_HOME:-$HOME/.grok}/feature-map.json
```

Vault layout (created on first run):

```text
<vault>/
  Index.md
  catalog.md
  projects/<slug>/Index.md
  projects/<slug>/<Canonical>.md
  patterns/Index.md
  patterns/<Canonical>.md
```

## Workflow

Follow phases in order for **one** repo. `all` repeats 1–7 per root.

### 0 — Config

Read [`references/first-run.md`](references/first-run.md).

- Missing config, missing vault path, or missing vault dir → wizard, then
  continue.
- Do not hardcode a machine path.

### 1 — Resolve project

- Repo root must exist and be a directory. Else **stop** (do not write
  `projects/`).
- Slug = directory name, lowercased, spaces → hyphens.
- If `projects/<slug>/Index.md` exists and its `repo_path` is a **different**
  repo, ask once for a new slug or reuse.
- Scratch: `$TMPDIR/feature-map-<slug>/`.

### 2 — Inventory

```bash
python3 "$INVENTORY" "$REPO_ROOT" > "$SCRATCH/inventory.json"
```

Non-zero exit → **stop**. No vault writes for this project.

Read `inventory.json`. Hits are hints, not the feature list.

### 3 — Coverage

Read [`references/coverage.md`](references/coverage.md). Assign every `files[]`
path. Incomplete if any path is unassigned.

Optional workers only when `file_count` is huge; they return classifications.
**This run** writes the vault.

### 4 — Slice + catalog

Read [`references/catalog.md`](references/catalog.md).

Slice coverage into features (contract §1). Match vault catalog. Pause on
unknowns with **one** table (proposed name, one-line why, aliases, sample
paths). Apply answers, then update vault `catalog.md`.

### 5 — Write notes

Read [`references/templates.md`](references/templates.md).

- New feature → new project note.
- Existing → refresh generated body; keep `## Human notes`.
- Gone → `status: stale`; keep file; do not regenerate from empty evidence.
- Create/update `patterns/<Canonical>.md`.
- Update `projects/<slug>/Index.md`, `patterns/Index.md`, vault `Index.md`.

Wikilinks: siblings `[[Canonical]]`; pattern `[[patterns/Canonical]]`; other
project `[[projects/<slug>/Canonical]]`.

### 6 — Next root (`all` only)

If this root failed, record the error and continue. Do not abort the batch.

### 7 — Handoff

Then **stop**.

```markdown
## Feature map
- Vault: <absolute path>
- Project: <slug> (<repo path>)   # or list for `all`
- Added: …
- Updated: …
- Stale: …
- Confirmed names: …
- Coverage: kept N · assigned N · not-a-feature N
- Failed roots: …                 # `all` only
```

## Anti-patterns

- Folding Impersonation into Auth
- A note per component, route, or helper
- Walking the tree by hand instead of the inventory script
- Writing notes before unknown confirmation
- Overwriting Human notes
- Deleting stale notes
- Hardcoding `~/Documents/Feature Maps`
- Committing the app repo or editing product code
- Opening a browser or capturing screenshots
- Smearing two projects into one `projects/` folder

## Red flags — stop and resume the protocol

- About to ship a 40-page “Auth” that includes impersonation, invites, and 2FA
- About to skip `node_modules`-adjacent app code that the inventory **kept**
- About to invent a canonical name that already has an alias in `catalog.md`
- About to rewrite a note that has Human notes without extracting that section
- About to run `all` roots in parallel

## Failure modes

| Situation | Action |
|-----------|--------|
| No config / vault path gone | First-run wizard; update config |
| Target not a project | Stop. No `projects/` write |
| Inventory fails | Stop this root. No partial vault writes |
| `python3` missing | Stop. Say the script needs Python 3 |
| `all` and one root fails | Record; continue |
| User rejects a proposed name | Use their name; proposal becomes an alias |
| Alias points at two canonicals | Ask; do not guess |
| Slug collision (same folder, different repo) | Ask once |
