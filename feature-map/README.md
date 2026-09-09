# feature-map

Grok skill. Maps a codebase into an Obsidian vault: **one note per feature**,
plus a shared **pattern** note so later projects can leapfrog.

## Install

Copy this folder to `~/.grok/skills/feature-map/` (or `$GROK_HOME/skills/feature-map/`).

Needs **Python 3**. Git is optional (used when the target is a git repo).

## Use

```text
/feature-map
/feature-map /path/to/repo
/feature-map all
```

First run asks where the vault should live (suggested: `~/Documents/Feature Maps`)
and optional project roots for `all`. That path is saved in
`~/.grok/feature-map.json` — never hardcoded.

Add the folder as a vault in Obsidian (Open folder as vault).

## What you get

```text
<vault>/
  Index.md
  catalog.md                 # canonical names + aliases
  projects/<slug>/<Feature>.md
  patterns/<Feature>.md      # cross-project comparison + default recipe
```

Each project feature note covers how it works, connections, a mermaid diagram,
key files, gotchas, and a **port to a new app** recipe. `## Human notes` is
yours on re-run.

## License

MIT
