# Eligibility

Canonical filter for implementable leaves in `.WCP/issues/`. Used by `/solve`, `/identify`, `/stat`, and `/tidy`.

Queue contract: [`../../docs/wcp-queue.md`](../../docs/wcp-queue.md). Claim and close: skill `water-cooler-protocol`. Do not call Linear.

---

## Inventory

List `*.md` under `.WCP/issues/open/`, `in-progress/`, and `blocked/`. Read frontmatter. Reclaim expired `in-progress` leases before selecting (player skill). Do not treat `done/` or `canceled/` as work to start.

`/solve today`: keep files whose `created` local date is today. Empty `created`: use `git log -1 --format=%cI -- <path>`.

---

## Eligible (2B)

Include:

- `status: open`
- `in-progress` only when the lease is expired (reclaim to `open` first) or `assignee` is this agent

Exclude:

- `done`, `canceled`, `blocked`
- `in-progress` with a future `lease_expires` and a different `assignee`

A `done` issue whose `commit` is an ancestor of local `dev` is finished for `/solve`. It is not a new implement target. A dependent whose `reason` names that id is no longer blocked: unblock it to `open` and leave `reason`.

---

## Blocked (2D)

A file in `blocked/` is not eligible. Read `reason`.

- `reason` names an id that is `done` or `canceled`: unblock to `open`, then it can be picked.
- `reason` names an id that is still `open` or `in-progress`: work that id first when it is in scope. Do not start the blocked file.
- `reason` names an out-of-scope id: skip the blocked file. Do not implement the out-of-scope blocker.
- `reason` is a human decision or a missing secret: skip.

Do not comment on ordinary skips. The file is the record.

---

## Parents (2E)

There is no epic. Implement the leaf file. If a body says it is packaging only and lists child ids, skip it and work those ids. Do not roll a parent up to `done` in place of the leaves.

---

## Scope filter

`/solve` leftover args name a cluster (`/solve design related issues`). This is not `SELECTION_PIN`.

1. `today` / `created today` / a goal to finish today's issues → `SCOPE = { kind: created, date: TODAY }`. Not an area named Today.
2. Else if the remaining words match a title or body in `open/` or `blocked/`: `SCOPE = { kind: area, query }`.
3. Several vague matches: ask once.
4. No match and the words are implementer constraints (`also`, `add`, `with`): no scope.
5. No match and the words were a cluster phrase: stop and ask. Do not drain the whole queue.

| `SCOPE` | User count | Mode |
| --- | --- | --- |
| set | none | `all` inside the scope |
| set | bare `N` | cap `N` inside the scope |
| unset | existing rules | default `1` / `N` / whole queue `all` |

`issue_in_scope`: `created` matches `created`'s local date; `area` matches title or body.

Out-of-scope blocker: skip the leaf. Do not implement the blocker.

`SELECTION_PIN` wins. Eligible = pin ∩ scope ∩ 2B ∩ unblocked. Do not refill outside the pin.

A preferred id (`0123` or `TEAM-123`) is the file whose `id` contains that number.

---

## Blocked quick reference

| Signal | Action |
| --- | --- |
| `blocked/` and blocker still open | Work the blocker if it is in scope; otherwise skip |
| `blocked/` and blocker is `done` or `canceled` | Unblock to `open`, then it is eligible |
| `blocked/` and blocker is out of scope | Skip |
| Live `in-progress` lease, other assignee | Skip |
| `done` with `commit` on local `dev` | Skip as implement target |
| Guidance says skip / abandoned platform | Cancel only at high confidence; otherwise skip |
| Packaging-only body | Work the listed child ids |
