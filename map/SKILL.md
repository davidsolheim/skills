---
name: map
description: >
  Use when the user runs /map, /map all, /map <path>, or /feature-map, or asks
  to map a project into Obsidian (features, tech stack, vendors), extract
  features from a codebase, build a cross-project catalog, or leapfrog
  implementations from past apps.
argument-hint: "[all | path]"
metadata:
  short-description: "Map projects into an Obsidian vault"
---

# /map — Codebase → Obsidian project map

Extract a **map** of a codebase into a shared Obsidian vault so later
projects can leapfrog. Today that is features plus a ranked vendor **Stack**;
more layers land on the same project Index. One note per feature per project,
plus a shared pattern note for the same capability across apps.

This skill is **read-only on the app repo**. It writes the Maps vault and
`${GROK_HOME:-~/.grok}/map.json` only.

## Operating contract

1. **Feature** = a capability you could ship, drop, or copy on its own, with
   a **real surface** (route, operator action, or API a client calls). Auth
   and Impersonation are both features; Impersonation `depends_on` Auth. A
   login button, `validateSession` helper, or `app/api/**/route.ts` is not a
   feature. **Admin** is a feature only if there is a distinct operator
   console; otherwise assign those files to the capabilities behind it.
   **Public API** only if it is a documented/partner/public HTTP API — not
   every Next handler. **Webhooks** only inbound/outbound product webhooks,
   not a string in a comment. **File uploads** only if a user/operator
   attaches a file. Never mint **Application** as a bucket. **Gameplay**
   only for actual games. Prefer split over stuffing Auth.
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
8. **Stack.** Every project Index has a ranked vendor table (most
   load-bearing first: “if this vanished, is the product dead?”). **Where**
   is code paths and env **names**, not only `package.json`. Vendors are not
   **features** — they are **stack hubs** under `stacks/<Vendor>.md` (who uses
   Stripe, Neon, Auth.js). Shape: [`references/templates.md`](references/templates.md).
9. **Leapfrog.** Read key source files, then write original prose. Token
   cost is not a constraint. Notes fail if a builder cannot tell what to
   copy. Rules: [`references/leapfrog.md`](references/leapfrog.md).
10. **Vault chrome.** Display names, home Index groups, catalog-as-list,
    tags, aliases, graph color groups:
    [`references/obsidian.md`](references/obsidian.md).

## Invocation

```text
/map
/map /path/to/repo
/map all
```

`/feature-map` is an alias of `/map`.

| Arg | Target |
|-----|--------|
| *(none)* | Current workspace (must be a project root) |
| path | That repo |
| `all` | Each `project_roots` entry, one at a time |

## Skill paths

```text
MAP_SKILL_DIR = directory containing this SKILL.md
INVENTORY     = $MAP_SKILL_DIR/scripts/inventory
FIRST_RUN_MD  = $MAP_SKILL_DIR/references/first-run.md
TEMPLATES_MD  = $MAP_SKILL_DIR/references/templates.md
CATALOG_MD    = $MAP_SKILL_DIR/references/catalog.md
COVERAGE_MD   = $MAP_SKILL_DIR/references/coverage.md
LEAPFROG_MD   = $MAP_SKILL_DIR/references/leapfrog.md
OBSIDIAN_MD   = $MAP_SKILL_DIR/references/obsidian.md
CONFIG        = ${GROK_HOME:-$HOME/.grok}/map.json
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
  stacks/Index.md
  stacks/<Vendor>.md
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
- Scratch: `$TMPDIR/map-<slug>/`.

### 2 — Inventory

```bash
python3 "$INVENTORY" "$REPO_ROOT" > "$SCRATCH/inventory.json"
# Optional bulk helper for huge `all` batches (still apply leapfrog.md to hubs):
# python3 "$MAP_SKILL_DIR/scripts/regen"
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

Read [`references/templates.md`](references/templates.md),
[`references/leapfrog.md`](references/leapfrog.md), and
[`references/obsidian.md`](references/obsidian.md).

- **Read** the 3–8 key files for each feature. Then write. Do not invent flow
  from paths alone.
- New feature → new project note (tags, aliases, display-quality prose).
- Existing → refresh generated body; keep `## Human notes`.
- Gone → `status: stale`; keep the file; do not regenerate from empty evidence.
- Create/update `patterns/<Canonical>.md` as a **comparison**, not a paste of
  project blurbs. Name one default donor.
- Update `projects/<slug>/Index.md` (**Stack** with `[[stacks/Vendor]]` links,
  code/env **Where** + feature list + display name). Incomplete without
  `## Stack`.
- Rebuild vault `Index.md` (grouped), `catalog.md` (pattern list), and
  `stacks/Index.md` (vendor list with app counts). Seed graph color groups
  if empty.

Wikilinks: `depends_on` / `unlocks` siblings only; pattern
`[[patterns/Canonical]]`; other project `[[projects/<slug>/Canonical]]`.

### 6 — Next root (`all` only)

If this root failed, record the error and continue. Do not abort the batch.

### 7 — Handoff

Then **stop**.

```markdown
## Map
- Vault: <absolute path>
- Project: <slug> (<repo path>)   # or list for `all`
- Added: …
- Updated: …
- Stale: …
- Confirmed names: …
- Stack: N vendors
- Coverage: kept N · assigned N · not-a-feature N
- Failed roots: …                 # `all` only
```

## Anti-patterns

- Folding Impersonation into Auth
- A note per component, route, or helper
- **Public API** / **Webhooks** / **File uploads** on every Next app
- **Application** as a junk bucket; **Gameplay** on a non-game
- Minting Stripe (or any vendor) as a **feature** canonical — vendors live under `stacks/`
- Dumping `package.json` into Stack; ranking by manifest order or alphabet
- README dump or “is a capability you could ship” as What it is
- Identical pattern-table rows; catalog cell with 40 wikilinks
- Linking every sibling feature (hairball graph)
- Flat alphabetical slug list as the vault home
- Walking the tree by hand instead of the inventory script
- Writing notes before unknown confirmation
- Overwriting Human notes
- Deleting stale notes
- Hardcoding `~/Documents/Maps`
- Committing the app repo or editing product code
- Opening a browser or capturing screenshots
- Smearing two projects into one `projects/` folder

## Red flags — stop and resume the protocol

- About to ship a 40-page “Auth” that includes impersonation, invites, and 2FA
- About to skip `node_modules`-adjacent app code that the inventory **kept**
- About to invent a canonical name that already has an alias in `catalog.md`
- About to rewrite a note that has Human notes without extracting that section
- About to run `all` roots in parallel
- About to skip `## Stack` or list eslint/lodash as vendors
- About to propose a vendor as a canonical because it is a vendor
- About to paste the README as What it is
- About to write a pattern table with identical Steal/Skip cells
- About to put forty project wikilinks in one catalog cell
- About to put `[[stacks/Stripe|Stripe]]` in a markdown table without escaping the alias pipe (`\|`)

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
