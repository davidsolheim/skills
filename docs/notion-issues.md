# Notion issue status

Use this only when `.wcp/tracker.md` says `tracker: notion`. wcp issues are always the record. If `tracker` is `none`, missing, or a different tool, do not follow this file.

The local queue stays `.wcp/issues/` ([`wcp-queue.md`](wcp-queue.md)). Write the file first. This file is the Notion procedure, and it runs after that write. If Notion fails, the file stands. Do not stop the git work. A later resync copies whatever this procedure missed.

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
| File moved to `done/` | `in-progress` |
| File moved to `deployed-dev/` | `in-review` |
| File moved to `deployed-main/` | `done` |
| File `canceled` | `canceled` |
| File `blocked` | `blocked` |
| File moved back to `open/` | `open` |

Notion has no separate status for `done/` or `deployed-dev/`. Use the statuses above. Notion `done` means the issue is in `deployed-main/`: the commit is on `origin/main`.

The skill that writes the issue file updates the row in the same turn, after the file write. A failed update does not roll back the file and does not block the next step. Fill `notion_page_id` and `notion_url` on the file when the row write succeeds.

## Resync

Walk `open/`, `in-progress/`, and `blocked/` whole. Walk `done/`, `deployed-dev/`, `deployed-main/`, and `canceled/` by the `YYYY/MM/DD` day folders, and include any older flat file still in those status directories. An `in-review/` file is `done`. Upsert each file by its issue id using the status table above. Read `dev` into **Dev SHA** and `main` into **Main SHA**. Running the walk twice is safe. There is no sync cursor. Do this when Notion was down, or when a file has an empty `notion_page_id` after a failed copy.

Do not move a Notion status backward from `done` except when the user says to reopen that issue.

## Ship

`/prb` and `/yeet` collect the ship set from `.wcp/issues/` and `git log origin/main..dev --pretty=%H%n%s`:

- a file whose `dev` sha is in that range
- a file whose id is the leading `0123` or `0123:` on a subject in that range

Skip `canceled`.

**On dev.** Immediately after `git push origin dev` succeeds: upsert each id. Set **Dev SHA** to that sha and Status to `in-review`. Write the same sha into `dev` on the issue file and move the file to `deployed-dev/`. Do not set Notion `done`.

**On main.** Immediately after the completion gate, set Status `done` and **Main SHA** to `origin/main`. Write that sha into `main` and move the file to `deployed-main/`. Fill **Deploy** when a production deployment id or URL is already known. Do not wait for a later session.

| Skill | Completion gate |
| --- | --- |
| `/prb` | The PR merged to `origin/main`. `--no-merge` does not set `done`. |
| `/yeet` | The production build of the merge SHA is Ready. Error or timeout does not set `done`. A later review-fix merge updates Main SHA. It does not delay this gate. |

If Notion fails, finish the git ship. Write `pr` on each shipped issue file. Leave `notion_page_id` empty when the row was not written. The next resync copies those files. Do not call Linear to replace that write.
