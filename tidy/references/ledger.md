# Tidy ledger and cooldown

Each processed issue gets a **Linear stamp** (source of truth across machines)
and a **local ledger** entry (fast skip on this machine).

Cooldown is **7 days** from the last stamp date unless `/tidy --force` or
`/tidy TEAM-123`.

## Linear stamp

Comment on the issue after you finish acting (or inspecting) it. **First line
exactly:**

```text
tidy-pass: YYYY-MM-DD · run <RUN_ID> · actions: <csv>
```

`actions` is a comma-separated list from:

`inspected` · `upgraded` · `retitled` · `related` · `status:In Review` ·
`status:Done` · `duplicate:TEAM-N` · `canceled` · `epic-rollup` · `needs-you`

Example:

```text
tidy-pass: 2026-08-18 · run 7f3a2c · actions: upgraded,retitled,related
```

One stamp per pass. Do not add a second stamp the same run. Optional one-line
evidence under the first line (sha, PR url) when status changed.

### Parse

Newest comment whose first line matches `^tidy-pass: (\d{4}-\d{2}-\d{2})`.
That date is `last_pass`. In cooldown if `today - last_pass < 7` days
(calendar dates, UTC or local consistently — use the machine’s local date).

Ignore older stamps. A newer stamp replaces the cooldown clock.

## Local ledger

Path:

```text
$HOME/.grok/tidy/ledgers/<workspace-id>.json
```

`workspace-id`: slug from `git config remote.origin.url` (host + path, no
`.git`, `/` → `--`). If no remote, use the absolute git common dir, then cwd.
Example: `github.com--acme--widgets`.

Never put tokens or issue *bodies* in the ledger. Ids + dates + action tags
only.

```json
{
  "team": "Example Team",
  "project": "Example Project",
  "remote": "github.com/acme/widgets",
  "updated": "2026-08-18",
  "issues": {
    "EX-123": {
      "last_pass": "2026-08-18",
      "run_id": "7f3a2c",
      "actions": ["upgraded", "retitled"]
    }
  }
}
```

Create `$HOME/.grok/tidy/ledgers/` if needed. Merge: overwrite only keys you
processed this run.

## Conflict

| Linear stamp | Ledger | Use |
|--------------|--------|-----|
| Present | anything | **Linear date** |
| Missing | present | Ledger date (this machine only) |
| Missing | missing | Due |

Do not invent stamps for skipped issues.

## Do not stamp

- Cooldown skips
- Live foreign claims / other-assignee In Progress
- Issues not in scope (`PINNED_ID` run)
- Terminal issues you only read as epic-child evidence
