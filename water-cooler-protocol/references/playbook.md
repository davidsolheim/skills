# Conflict and red-test playbook

Verbs are in `../SKILL.md`. This file is the decision table. Ticket claim, renew, reclaim, close, cancel, and block are the Issues section of that skill. Do not `wcp acquire` a file under `.WCP/issues/`.

## Name

1. `wcp name <id> --json`, then export `WCP_AGENT` and `WCP_NAME_TOKEN`.
2. `name_taken` → pick a different id (`issue-123-b`). You do not already hold this one.
3. `already_named` → keep the name in the message. Do not rename.

## Before acquire

1. Write or append the test. Do not acquire the test file.
2. The test line names the existing path: `WCP <id>: <path> …`.
3. A path that was not in the tree when the run started: write it. Do not acquire.
4. A path that was already in the tree: `acquire --test <test-file>`.

## acquire errors

| error | do |
|---|---|
| `no_test` | write the test line that names this path, then acquire. Do not claim the test file |
| `test_file` | you tried to claim a test. Write it directly |
| `new_file` | you tried to claim a new file. Write it directly |
| `conflict` | read `existing.doing` and `existing.scope` |
| `agent_busy` | finish or release the path you already hold |

On `conflict`:

1. Same symbol or same intent, lease live → retarget to another file from `arch`.
2. Same intent, lease idle (`expired`) → `overtake --test <your-test>` only to finish their burst.
3. No overlap → pick another path from `arch`. Re-read disk before a later acquire. Do not rewind their hunks.

## write-ok

| result | do |
|---|---|
| `claim: test` | write the test. No lease |
| `claim: new_file` | write the file. No lease |
| `no_lease` / `expired` | this file existed at run start. Write the test, then acquire |
| `wrong_path` | you hold a different file; release or finish that burst first |
| `wrong_branch` | stop. You are not on `run.branch` |
| `drift` | stop. look. retarget. never rewind |
| `no_run` | stop. Human must `wcp start --arch` |

## Tests red

Run tests you added (`WCP <id>`). If someone else's test is red, they are mid-edit or you collided. Do not edit their source to pass yours. Re-look. If you drifted them, retarget; do not restore their file to an older blob.
