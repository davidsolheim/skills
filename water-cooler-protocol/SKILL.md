---
name: water-cooler-protocol
description: >
  The work queue in .wcp/issues/. Many coding agents share one local checkout.
  An issue in in-progress means that agent is editing it now. There is no daemon.
  Load when writing on local dev, when coordinating parallel agents, or when
  the user runs /wcp. Skill-integration contract: ~/.grok/skills/docs/wcp.md.
---

# wcp

You share this checkout with other agents. The issue files are the protocol. Edit them. There is no `wcp` command, no daemon, and no lease to acquire.

If this repo contains `PROTOCOL.md`, that file is the same rules. Other skills follow [`../docs/wcp.md`](../docs/wcp.md). The queue folder is `.wcp/`.

## Folders

```
.wcp/issues/open/                          backlog. does not block a commit
.wcp/issues/in-progress/                   you are editing this, on this machine
.wcp/issues/done/YYYY/MM/DD/               writing is finished. not pushed
.wcp/issues/deployed-dev/YYYY/MM/DD/       commit is on origin/dev
.wcp/issues/deployed-main/YYYY/MM/DD/      commit is on origin/main
.wcp/issues/blocked/                       still wanted. do not take it. reason is set
.wcp/issues/canceled/YYYY/MM/DD/           will not be done. reason is set
```

`open/`, `in-progress/`, and `blocked/` stay flat. The other four go under `YYYY/MM/DD` from `created`. The filename is `YYYYMMDDThhmmZ-<id>-<slug>.md` and stays the same when you move it. Set `status`, then move the file.

An issue sitting in `in-review/` is finished writing. Move it to `done/`.

## Run

1. Look in `done/`. A commit already on `origin/dev` moves to `deployed-dev/` and you write the sha in `dev`. A commit already on `origin/main` moves to `deployed-main/` and you write the sha in `main`.
2. Read `in-progress/`. Those `files` lists are the paths the other agents are in. Work around them.
3. An `in-progress/` issue whose agent is gone goes back to `open/`. Clear `assignee`.
4. Take one issue from `open/` that does not use those paths. Move it to `in-progress/`. Set `assignee` to your name. Read it back. If `assignee` is not you, stop.
5. Add each path to `files` as you write it.
6. When the writing matches `acceptance`, move the issue to `done/`.
7. If `in-progress/` still has anything, or you still have another task, stop. Do not commit. Do not push.
8. If `in-progress/` is empty and you have no further task, commit the tree, push `dev` to `origin/dev`, write that sha into `dev` on each `done/` issue from that commit, and move those issues to `deployed-dev/`.

`open/` does not block the push. Only `in-progress/` means someone is still editing.

Do not reset, checkout, or stash. Another agent's uncommitted work stays. Commit `.wcp/issues/` and `.wcp/tracker.md`. Do not commit any other path under `.wcp/`.

If you edit a file that already has uncommitted changes, find every issue in `in-progress/` or `done/` whose `files` list includes that path. Read that issue and the other paths in its `files`. Keep the behavior its `acceptance` describes. Your edit builds on those changes. Do not revert them, and do not leave that acceptance broken.

## External tracker

wcp issues are always the record. An external tracker is optional.

Read `.wcp/tracker.md`. `tracker: none`, or a missing file, means do not call Notion, Linear, Jira, or anything else. When the user is here and the file is missing, ask once which tracker they want. `none` is a complete answer. Write the file and do not ask again. If you cannot ask, leave it missing and keep working.

```yaml
---
tracker: none
url:
---
```

`tracker` may be `notion`, `linear`, `jira`, or another name the user gave. `url` is that board or project.

When `tracker` is not `none`, mirror every create, status move, `files` change, and `dev` or `main` sha to that tool after the file write. For `notion`, follow [`../docs/notion-issues.md`](../docs/notion-issues.md). A failed mirror does not change the file. Do not keep a second queue.

## Issue file

```yaml
---
id: 0123
title: Rotate refresh token on session mint
status: open
assignee:
files: []
acceptance:
reason:
dev:
main:
created: 2026-10-01T12:02:00Z
session: 2026-10-01T12:00:00Z
---
```

The body is the spec. `dev` and `main` stay empty until those commits exist. Search `blocked/` and `canceled/` before filing the same work again. `reason` is required to block or cancel. One change that can ship on its own is one file.

A later run moves an issue from `deployed-dev/` to `deployed-main/` only after that commit is on `origin/main`, and writes the sha into `main`.
