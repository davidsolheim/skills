# map

Grok skill. Maps a codebase into an Obsidian vault: **one note per feature**,
a ranked **Stack** of vendors on the project Index, plus a shared **pattern**
note so later projects can leapfrog. More layers join the same project Index.

## Install

Copy this folder to `~/.grok/skills/map/` (or `$GROK_HOME/skills/map/`).

Needs **Python 3**. Git is optional (used when the target is a git repo).

## Use

```text
/map
/map /path/to/repo
/map all
```

`/feature-map` is an alias of `/map`.

First run asks where the vault should live (suggested: `~/Documents/Maps`)
and optional project roots for `all`. That path is saved in
`~/.grok/map.json` — never hardcoded.

Add the folder as a vault in Obsidian (Open folder as vault).

## What you get

```text
<vault>/
  Index.md
  catalog.md                 # feature pattern hubs
  stacks/Index.md            # vendor hubs (who uses Stripe, Neon, …)
  stacks/<Vendor>.md
  projects/<slug>/Index.md   # ranked Stack + feature list
  projects/<slug>/<Feature>.md
  patterns/<Feature>.md      # cross-project comparison + default recipe
```

Each **project Index** has a ranked **Stack** (code/env evidence, most
load-bearing first) plus features. Feature notes are leapfrog write-ups
(real flow, mermaid, port recipe). Pattern notes **compare** implementations
and name one donor. The vault home groups products / sites / tools; the
catalog is a list of pattern hubs, not a giant table. `## Human notes` is
yours on re-run. Token cost is not a constraint — thoroughness is.

## License

MIT
