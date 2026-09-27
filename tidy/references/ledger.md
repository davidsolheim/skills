# Tidy ledger and cooldown

Each processed issue gets a **local ledger** entry. That entry is the cooldown clock. Do not post a Linear comment. Do not call Linear.

Cooldown is **7 days** from `last_pass` unless `/tidy --force` or `/tidy 0123`.

## Stamp

After you finish acting on (or inspecting) an issue, merge one object into the ledger. `actions` is a list from:

`inspected` · `upgraded` · `retitled` · `related` · `status:in-review` · `status:done` · `duplicate:0123` · `canceled` · `needs-you`

One stamp per issue per pass. If this `run_id` is already on that id today, do not write a second stamp.

## Local ledger

Path:

```text
$HOME/.grok/tidy/ledgers/<workspace-id>.json
```

`workspace-id`: slug from `git config remote.origin.url` (host + path, no `.git`, `/` → `--`). If no remote, use the absolute git common dir, then cwd.
Example: `github.com--acme--widgets`.

Never put tokens or issue *bodies* in the ledger. Ids + dates + action tags only.

```json
{
  "remote": "github.com/acme/widgets",
  "updated": "2026-08-18",
  "issues": {
    "0123": {
      "last_pass": "2026-08-18",
      "run_id": "7f3a2c",
      "actions": ["upgraded", "retitled"]
    }
  }
}
```

Create `$HOME/.grok/tidy/ledgers/` if needed. Merge: overwrite only keys you processed this run.

## Due

| Ledger `last_pass` | Use |
|--------------------|-----|
| Present and younger than 7 days | Skip, unless `--force` or this id is pinned |
| Present and 7 days or older | Due |
| Missing | Due |

Use the machine’s local date. Do not invent stamps for skipped issues.

## Do not stamp

- Cooldown skips
- Live foreign claims
- Issues not in scope (`PINNED_ID` run)
- `done/` and `canceled/` files you only read as evidence
