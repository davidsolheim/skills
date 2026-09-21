# Conflict and red-test playbook

Verbs are in `../SKILL.md`. This file is the decision table.

## acquire → conflict

1. Read `existing.doing` and `existing.scope` (and `expired`).
2. Overlap with your symbol/intent, lease live → wait or pick another file from `arch`.
3. Overlap, lease idle (`expired`) → `overtake` only to finish their burst.
4. No overlap → wait for release, then acquire; re-read disk; do not rewind their hunks.

## write-ok failures

| error | do |
|---|---|
| `no_lease` / `expired` | acquire again |
| `wrong_path` | you hold a different file; release or finish that burst first |
| `wrong_branch` | stop. You are not on `run.branch` |
| `drift` | stop. look. wait or retarget. never rewind |
| `no_run` | stop. Human must `wcp start --arch` |

## Tests red

Run tests you added (`WCP <id>`). If someone else's test is red, they are mid-edit or you collided. Do not edit their source to pass yours. Re-look. If you drifted them, retarget; do not restore their file to an older blob.
