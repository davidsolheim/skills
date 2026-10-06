# wcp in Teton skills

Many coding agents share one local `dev` checkout. The issue files under `.wcp/issues/` are the protocol. There is no daemon and no `wcp` binary.

Spec: [PROTOCOL.md](https://github.com/davidsolheim/water-cooler-protocol/blob/dev/PROTOCOL.md). Player skill: `water-cooler-protocol`. Queue layout: [`wcp-queue.md`](wcp-queue.md).

The queue folder is `.wcp/`.

`/solve`, `/issues`, `/issue`, `/identify`, `/start`, `/prb`, `/yeet`, `/project-review`, `/walk`, `/tidy`, `/human-copy`, `/vercel-flags`, TypeSafe, composition refactors, `/implement`, and `/execute-plan` follow this file. Do not invent a second way to claim a file.

## What occupancy is

An issue in `.wcp/issues/in-progress/` means an agent is editing that work on this machine. Read its `files` list and work around those paths. There is no file lease, no heartbeat, and no test-comment gate.

`open/` is the backlog. It does not block a commit. `blocked/` and `canceled/` are side doors. Read `reason`. Do not take them.

Do not reset, checkout, or stash. Another agent's uncommitted work stays.

If you edit a file that already has uncommitted changes, find every issue in `in-progress/` or `done/` whose `files` list includes that path. Read that issue and the other paths in its `files`. Keep the behavior its `acceptance` describes. Your edit builds on those changes. Do not revert them, and do not leave that acceptance broken.

## Who does what

| Actor | Duty |
| --- | --- |
| Any agent on this checkout | Follow the player skill. Move the issue you take to `in-progress/`. Move it to `done/` when the writing is finished. |
| The agent that finds `in-progress/` empty and has no further task | Commit the tree, push `dev` to `origin/dev`, and move those `done/` issues to `deployed-dev/`. |
| A later run | Move a `done/` issue whose commit is already on `origin/dev` to `deployed-dev/`. Move an issue to `deployed-main/` only after that commit is on `origin/main`. |
| `/issue` `/issues` `/project-review` `/walk` `/tidy` `/identify` `/stat` | Read and write `.wcp/issues/` ([`wcp-queue.md`](wcp-queue.md)). Mirror to the external tracker only when `.wcp/tracker.md` names one. |

One checkout, one queue. Do not share one queue across worktrees.

## Run

1. Look in `done/`. Move an issue forward when its commit is already on `origin/dev` or `origin/main`.
2. Read `in-progress/`.
3. Take one issue from `open/` whose files do not overlap. Move it to `in-progress/`.
4. When the writing is finished, move it to `done/`.
5. If anything is still in `in-progress/`, or you still have a task, stop. Do not commit.
6. If `in-progress/` is empty and you have no further task, commit, push `origin/dev`, and move those issues to `deployed-dev/`.

Commit `.wcp/issues/` and `.wcp/tracker.md`. Do not commit any other path under `.wcp/`.

## External tracker

wcp issues are always created and managed. Ask the user once, when `.wcp/tracker.md` is missing, which external tracker they want. `none` is a complete answer. Record it:

```yaml
---
tracker: none
url:
---
```

`notion`, `linear`, `jira`, or another name the user gives are all valid. `url` is that board. A missing file or `none` means do not call an external tracker.

When a tracker is named, mirror every create, status move, `files` change, and `dev` or `main` sha after the file write. Notion uses [`notion-issues.md`](notion-issues.md). A failed mirror does not change the file.
