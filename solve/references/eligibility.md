# Eligibility (leaves, blocked, epics)

Canonical filter for **implementable unblocked leaves**. Used by `/solve` and
`/identify`. Do not invent a second blocked/claim policy.

Claim / CAS details: [`multiplayer-linear.md`](multiplayer-linear.md).

**Identify inventory is read-only** on these rules: expand to a leaf, never
recommend a parent shell, **do not** rollup-Done or comment while picking.
`/solve` **does** rollup when all children are terminal (see Epic rollup).

---

## Linear inventory fetch

Used by `/identify` Phase 2, `/solve` Phase 2A / drain gate, `/tidy`
Phase 2, and `/stat` Phase 2. **Do not** `list_issues` with only
`team` + `project`. That returns Done/Canceled and truncates huge payloads.

1. `list_issue_statuses` for the team.
2. Keep statuses whose **type** is `backlog`, `unstarted`, or `started`
   (covers 2B plus In Review / Blocked so 2B–2D can drop them). Drop types
   `completed`, `canceled`, `duplicate`.
3. For each kept status **name**, `list_issues` with `team` + `project` +
   `state` (that name). `state` accepts type, name, or ID — use the **name**
   from step 1 so Todo and Backlog are both fetched. Parallelize per status.
4. Page each state until `hasNextPage` is false. A reasonable `limit` (50–250)
   is fine; do not raise it to compensate for a missing `state` filter.
5. Inventory `fields` only: `id`, `title`, `priority`, `url`, `status`,
   `statusType`, `labels`, `assignee`, `assigneeId`, `parentId`, `team`,
   `project`, `projectMilestone`. **Omit `description`.**
6. `get_issue` (full body + `includeRelations`) and `list_comments` only for
   candidates that need 2D/2E (`blockedBy`, `claimed-by`), that enter rank /
   propose / claim, or that `/tidy` marks **due**.

`list_issues` with `parentId` (epic expand) is a targeted child fetch — do not
add a project-wide `state` filter there; 2E needs terminal children too.

---

## Done vs In Review vs on `dev`

Linear status is **not** git. Two different gates:

| Status | Means | `/solve` may mark it? | Terminal for epic rollup? |
|--------|--------|------------------------|---------------------------|
| **Done** | On `main` (via `/prb`) | **No** | Yes |
| **In Review** | *Claimed* to be on `dev`, waiting on `/prb` → `main` | Yes (closeout) | **No** |
| Backlog / Todo / In Progress | Not shipped to `dev` unless git says otherwise | In Progress while working | No |

**In Review is a hypothesis, not proof.** A prior `/solve` is *supposed* to set it only after merge to local `dev` (and `origin/dev` when that ship path ran). Status can be stale, optimistic, or left over from a run that never landed the commit. Before skipping an In Review leaf **or** treating it as a satisfied `blockedBy`, run the [On-dev check](#on-dev-check-in-review).

`/identify` uses the same check so it does not recommend honest In Review work as “next,” and does recommend a lying In Review as “complete this.”

### On-dev check (In Review)

Run when a **candidate** or a **`blockedBy` target** is In Review (also if In Progress comments claim `shipped origin/dev` / merged to `dev`).

1. `git fetch origin` if this phase has not already.
2. Collect SHAs from completion comments (`origin/dev` SHA, `merged into local \`dev\``, `shipped origin/dev @ sha`).
3. Also search `git log --oneline dev` (local `dev`) for the issue identifier in the subject (`POR-332`, `TEAM-123`).
4. **`on_dev` is true** if any collected SHA is an ancestor of local `dev` (`git merge-base --is-ancestor <sha> dev`) **or** a local `dev` commit subject contains the issue id.
5. Local `dev` is **sufficient**. Sequential `/solve` often does not push; do not require `origin/dev` or `main`. If `origin/dev` exists, you may also record whether the SHA is there, but absence of `origin/dev` does not fail the check.
6. **`on_dev` is false** when there is no matching SHA/commit. Treat the Linear status as stale.

Do not invent “it is probably on dev because the ticket is In Review.”

---

## Eligible states (2B)

Include:

- Backlog, Todo, Ready, Triage, Unstarted, and team equivalents
- **In Progress** only if **unclaimed** (no live foreign `claimed-by:` comment
  < 60 min) **or** already claimed by **this run**
- **In Review** only if **`on_dev` is false** — complete the leaf (investigate,
  finish remaining work, verify, merge local `dev`, keep/set In Review). Do
  not skip a lying In Review.

Exclude:

- Done, Canceled, Cancelled, Duplicate, Completed
- **In Review and `on_dev`** — already solved for `/solve`; waiting on `/prb`
  to merge `main`. Not a new implement target. **Still not terminal.**
- In Progress **claimed by another run** (foreign live `claimed-by:`)
- In Progress assigned to **someone else** (assignee is not this Linear user)
- Issues whose only remaining work is explicitly “wait for external X” with X
  unavailable

When an In Review leaf fails `on_dev`, pick it in lowest-number / guidance
order like any other eligible leaf. Completing it **unblocks** dependents that
listed it in `blockedBy`.

---

## Blocked (2D)

A `blockedBy` relation **does not block** when that target is **satisfied for
unblocking**. Satisfied:

- Done / Canceled / Cancelled / Duplicate / Completed
- **In Review (or shipped-to-dev In Progress) and `on_dev`**

Not satisfied (the target still blocks **or** becomes the work):

- Backlog / Todo / Triage / In Progress without `on_dev`
- **In Review and not `on_dev`** — do **not** skip the dependent forever and
  do **not** wait for `main`. The **unsatisfied blocker** is the implementable
  leaf: complete it first, then the dependent is unblocked.

Treat as **blocked** (skip this leaf, try next / or pick the blocker) if any of:

- Open `blockedBy` to a **not-satisfied** issue (see above). If that blocker is
  itself eligible (including stale In Review), **work the blocker**, do not
  discard the dependent’s epic. When `SCOPE` is set, only work that blocker if
  it is also in scope ([Scope filter](#scope-filter-milestone--group--area)).
- State/label clearly means Blocked
- Description/comments require missing credentials, a human product decision,
  or an unpaid external dependency you cannot resolve in-repo
- Duplicate of an open issue (work the original if it is the canonical leaf)
- Guidance marks the leaf `skip` / full obsolete platform (do not implement as
  written)

If blocked only by In Review-on-`dev`, **it is not blocked.** Continue to that
leaf (example: POR-332 In Review on local `dev` → POR-333 is eligible).

If blocked, record why briefly and continue. **No** Linear comment on ordinary
blocked skips.

---

## Epic / parent expansion (2E)

**Never** implement a parent/epic as the coding target when it has children.
Expand into a child instead.

### Detect parent / epic

Treat as a **parent (epic)** if **any** of:

1. `list_issues` with `parentId` set to this issue returns **one or more**
   children (canonical — page if needed).
2. Labeled **Epic** (or team equivalent) **and** has children under (1).
3. Type/name clearly indicates Epic **and** has children under (1).

Epic label/type with **zero** children → not expandable; treat as a normal leaf
(or too-thin placeholder).

### Expand

```text
function resolve_work_issue(candidate):
  if candidate is blocked → return skip (no Linear comment), try next
  children = list_issues(parentId=candidate)   # all pages
  if children is empty:
    return candidate   # leaf
  eligible_children = children that pass 2B and 2D
  if SCOPE is set: keep only issue_in_scope children
  sort eligible_children by TEAM-(\d+) ascending
  if eligible_children is empty:
    if every child is terminal (Done / Canceled / Cancelled / Duplicate / Completed):
      if caller is /solve:
        comment on epic: all children complete → marking epic Done
        set epic state Done
      # Identify: skip only; no rollup write
      return skip
    else:
      record "epic <id> has no eligible children (not all terminal)"
      return skip (no comment)
  return resolve_work_issue(eligible_children[0])
```

- One expansion yields one leaf for this slot.
- Prefer the **lowest-numbered eligible unblocked child of this epic**, not the
  globally lowest issue, once the epic is the candidate.
- Nested parents: recurse until a leaf.
- While a child is still open: do not claim or Done the parent; work the leaf.
- Preferred id is an epic: expand to its lowest eligible child.
- Preferred id is a child: use that child if eligible (do not expand upward).

### Epic rollup (`/solve` only)

When **every** child is terminal (Done / Canceled / Cancelled / Duplicate /
Completed). **In Review is not terminal** even when `on_dev` — a child on
local/`origin` `dev` keeps the epic open until `/prb` marks it Done on `main`.
In Review-on-`dev` children **do** count as satisfied blockers for **siblings**
(2D); they just do not roll the epic to Done.

1. Comment: all children complete; closing epic as rollup (list child ids if short).
2. Set epic **Done**.
3. Do not implement or git the epic shell.
4. Do not count the epic toward `/solve N`.

Identify must not perform this write during inventory. `/tidy` may rollup on
its own pass.

---

## Scope filter (milestone / group / area)

Used by `/solve` when leftover args name a cluster (e.g. `/solve design related
issues`). Optional for `/identify` / `/stat` area filters.

This is **not** `SELECTION_PIN`. Pin is an ordered closed id list. Scope is a
**recomputed membership** each inventory (new matching leaves can enter; issues
that leave the milestone/label drop).

### Resolve `SCOPE` (after team + project)

Inputs: `REST` = leftover tokens after `--effort`, `--concurrency`, `fast`,
count (`all` / bare `N`), and `TEAM-\d+`.

1. Detect **cluster phrasing** if `REST` matches (case-insensitive):
   `related (issues|tickets|leaves)`, `(issues|tickets) in`, `milestone`,
   `label:`, `labeled`.
2. Strip filler tokens: `related`, `issues`, `tickets`, `leaves`, `the`, `a`,
   `an`, `group`, `milestone`, `labeled`, `label`, `for`, `in`.
3. `QUERY` = remaining tokens joined with a space. Empty `QUERY` → no `SCOPE`.
4. `list_milestones` for the project.
   - Match if milestone **name** equals `QUERY`, contains `QUERY`, or `QUERY`
     contains the name (name length ≥ 3), case-insensitive.
   - **One** match → `SCOPE = { kind: milestone, id, name }`.
   - **Several** → ask once with the names; do not guess.
5. Else `list_issue_labels` for the team. Same matching on label **name**.
   One match → `SCOPE = { kind: label, name }`. Several → ask once.
6. Else if cluster phrasing **or** `QUERY` matches any open issue title, label,
   or identifier: `SCOPE = { kind: area, query: QUERY }`.
7. Else if `REST` looks like implementer constraints (starts with `also` /
   `don't` / `do not` / `with` / `using` / `include` / `add` / `implement` /
   `make sure`) **and** no milestone/label hit: **no** `SCOPE`; `REST` is extra
   constraints (existing default count **1**).
8. Else: **stop** and ask whether to re-run unscoped. Do not silently drain the
   whole project. Do not treat a failed cluster phrase as implementer constraints.

After a milestone/label hit, leftover `REST` beyond the matched name (e.g.
`also add a unit test`) is extra implementer constraints.

### Implicit count

| `SCOPE` | User count token | `SOLVE_COUNT_MODE` |
|---------|------------------|--------------------|
| set | none | **`all`** (drain this scope) |
| set | `all` | **`all`** (this scope) |
| set | bare `N` | **`N`** (cap inside this scope) |
| unset | existing rules | default **1** / `N` / project-wide `all` |

### Membership (`issue_in_scope`)

Apply to the open inventory **before** pick / S0 / drain gate / fast F6 refill.

| `kind` | In scope when |
|--------|----------------|
| `milestone` | `projectMilestone` id/name is `SCOPE`, **or** `parentId` is an in-scope issue (untagged children of a scoped epic), **or** this id is `parentId` of in-scope issues (epic shell — expand only, never implement) |
| `label` | any issue label name matches `SCOPE` (case-insensitive) |
| `area` | title, labels, or identifier contains `QUERY` (case-insensitive) |

Two-pass for milestone parent/child. Do not `get_issue` the whole board for this.

### Out-of-scope blockers

If an in-scope leaf has `blockedBy` on an **unsatisfied out-of-scope** issue:
skip the leaf as blocked. Do **not** implement the out-of-scope blocker (that
would expand the drain past the user’s cluster). If the blocker is in-scope,
2D still says work the blocker first.

### Pin vs scope

`SELECTION_PIN` **wins**: eligible = pin ∩ scope ∩ (2B + unblocked + expanded
leaves). Do not refill outside the pin. If only scope is set, refill from
matching inventory each cycle; do not pick outside `SCOPE`.

---

## Selection pin (optional)

When a caller injects **`SELECTION_PIN`** (ordered identifier list) — `/identify`
does this after approve:

1. Eligible set is **only** `SELECTION_PIN` ∩ (2B + unblocked + expanded leaves).
   If `SCOPE` is also set, intersect with `issue_in_scope`.
2. Walk **pin order**. Skip missing / ineligible / guidance-skip (report skip).
3. **Do not** pick any leaf outside the pin.
4. **Do not** refill from outside the pin (`all` drain and fast F6 included).
5. If S0 ran, guidance skip/cancel still applies to pinned ids.

`/solve 1 TEAM-123` from Identify sets `SELECTION_PIN = [that id]` and
`preferred issue` to that id. Reuse Identify’s `RUN_ID` so the identify
`claimed-by:` is this run, not foreign.

---

## Blocked quick reference

| Signal | Action |
|--------|--------|
| `blockedBy` **not** `on_dev` / not Done | Work the **blocker** if it is eligible **and in `SCOPE`** (when set); otherwise skip this leaf (no comment) |
| `blockedBy` out of `SCOPE` | Skip this leaf. Do **not** implement the out-of-scope blocker |
| `blockedBy` In Review **and** `on_dev` | **Not blocked.** Implement this leaf |
| State/label Blocked | Skip (no comment) |
| Needs missing secrets / human decision | Skip (no comment) unless already claimed → then Blocked + comment |
| In Progress + other assignee | Skip (no comment) |
| Done/Canceled/Duplicate | Skip (no comment) |
| In Review **and** `on_dev` | Skip as implement target (waiting on `/prb` / `main`). Still unblocks dependents |
| In Review **and not** `on_dev` | **Implement** — status is stale; complete onto local `dev` |
| Guidance full-obsolete / abandoned platform | Skip implement; `/solve` may comment + Cancel/Duplicate when high confidence. Identify: drop from `ELIGIBLE` only |
| Parent/epic with eligible children | Expand → lowest eligible child |
| Epic, all children terminal | `/solve`: comment + Done (rollup). Identify: skip, no write |
| Epic, no eligible children, some non-terminal | Skip epic (no comment) |
