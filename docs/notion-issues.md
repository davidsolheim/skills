# Notion issue status

External status for `/issue`, `/issues`, `/solve`, `/identify`, `/stat`, `/tidy`, `/project-review`, `/walk`, `/start`, `/prb`, and `/yeet`.

The local queue stays `.WCP/issues/` ([`wcp-queue.md`](wcp-queue.md)). Claim, lease, and review stay in those files. This file is the only Notion procedure. Do not call Linear.

One issues database per git repo. The database title is the repo slug. The database is identified by the origin URL.

## Identity

```bash
git remote get-url origin
```

Normalize that URL:

- `git@github.com:owner/repo.git` → `https://github.com/owner/repo`
- drop `https://user:token@` credentials
- drop a trailing `.git` and a trailing slash
- lowercase the host

Slug = the last path segment, lowercased. `https://github.com/owner/Water-Cooler-Protocol` → `water-cooler-protocol`.

The database **description** is that normalized URL, exactly. Every row's **Repo** property is the same URL.

## Find or create

1. `notion-get-tool-access` with `{}` when this connection's access is not already in context. If `ai_search` is available, search with it. Otherwise use `notion-search`.
2. Search the slug and the normalized URL.
3. `notion-fetch` each database candidate. Reuse the one whose description equals the URL. A title match with a different URL is a different repo.
4. No match: search for a page whose body contains that URL. Create the database under that page (`parent.page_id`). No such page: omit `parent` so Notion creates a private database, and say that it is private.
5. `notion-create-database`:

```text
title: <slug>
description: <normalized origin URL>
schema: CREATE TABLE ("Name" TITLE, "Issue" RICH_TEXT, "Status" SELECT('open':gray, 'in-progress':blue, 'in-review':yellow, 'done':green, 'canceled':red, 'blocked':orange), "Priority" SELECT('critical':red, 'high':orange, 'normal':yellow, 'low':gray), "Repo" URL, "PR" URL, "Dev SHA" RICH_TEXT, "Main SHA" RICH_TEXT, "Deploy" RICH_TEXT)
```

6. `notion-fetch` the database before any row write. Use the data source URL from `<data-source>` and the property names from that schema. If a required column is missing on a reused database, `notion-update-data-source` `ADD COLUMN` for that column only.

Discover each tool with `search_tool` before `use_tool`. Do not invent parameter names.

## Rows

Match a row by **Issue** = the queue id (`0123`) inside this database.

- Missing row: `notion-create-pages` with `parent.data_source_id`.
- Existing row: `notion-update-page` with `command: update_properties`.

Properties:

| Property | Value |
| --- | --- |
| Name | issue title |
| Issue | `0123` |
| Status | the table below |
| Priority | `low` `normal` `high` `critical` |
| Repo | normalized origin URL |
| PR | pull request URL when the ship has one |
| Dev SHA | `origin/dev` after that push |
| Main SHA | `origin/main` after that ship |
| Deploy | production deployment id or URL when known |

Do not put tokens, connection strings, or secret values in any property or page body.

## Status

| When | Notion Status |
| --- | --- |
| File created in `open/` | `open` |
| File moved to `in-progress/` | `in-progress` |
| File moved to `in-review/` | `in-review` |
| File `done` and its `commit` is not on `origin/main` | `in-review` |
| File `canceled` | `canceled` |
| File `blocked` | `blocked` |
| File moved back to `open/` | `open` |
| `/prb` or `/yeet` finishes the ship below | `done` |

`done` on Notion means the work is on `origin/main` and that ship skill's completion gate passed. A reviewer setting the local file to `done` does not set Notion to `done`.

The skill that writes the issue file updates the row in the same turn, after the file write. Do not call Notion during a source-file lease. A worker does not call Notion. The orchestrator, the filing skill, or the ship skill does.

Do not move a Notion status backward from `done` except when the user says to reopen that issue.

## Ship

`/prb` and `/yeet` collect the ship set from `.WCP/issues/` and `git log origin/main..dev --pretty=%H%n%s`:

- a file whose `commit` is in that range
- a file whose id is the leading `0123` or `0123:` on a subject in that range

Skip `canceled`.

**On dev.** Immediately after `git push origin dev` succeeds and the PR exists: upsert each id. Set **Dev SHA** to `origin/dev` and **PR** to the PR URL. If Status is `open` or `in-progress`, set `in-review`. Do not set `done`.

**On main.** Immediately after the completion gate, set Status `done` and **Main SHA** to `origin/main`. Fill **Deploy** when a production deployment id or URL is already known. Do not wait for a later session.

| Skill | Completion gate |
| --- | --- |
| `/prb` | The PR merged to `origin/main`. `--no-merge` does not set `done`. |
| `/yeet` | The production build of the merge SHA is Ready. Error or timeout does not set `done`. |

If Notion fails, finish the git ship and list the ids whose status was not written. Do not call Linear to replace that write.
