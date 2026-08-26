---
name: stat
description: >
  Use when the user runs /stat, says "stat", "board status", "project status",
  "what's open", "show issues that need to be resolved", "what's left to
  solve", or wants a read-only briefing of this repo's Linear project sorted
  most urgent to least. Also /stat N or area words. Do not use for session
  /status (auth/model/context) or for picking a solve batch (/identify).
argument-hint: "[N] [area…]"
---

# /stat — Open issues, urgent first

Read **this repo’s Linear project**. List **every issue that still needs
resolution**. Sort **most urgent → least**. Then **stop**.

This skill does **not** implement, claim, tidy, upgrade, or start `/solve` /
`/identify`.

## Operating contract

- **Whole open board.** Done / Canceled / Duplicate / Completed are resolved;
  do not list them. Blocked, claimed, In Progress, In Review, and epic shells
  still need resolution — **include them**.
- **One ranked list**, not a 2–4 ticket batch. Do not apply Identify size cut
  or overlap cut.
- **Rank** = Linear **priority**, then **user-facing impact** (`U`), then
  identifier number. Keys: `$IDENTIFY_SKILL_DIR/references/ranking.md`
  (Sort keys + Priority + User-facing impact **only**).
- **Read-only on Linear.** No comments, assignee, or state changes.
- **Secrets:** never print tokens, env values, or Doppler secrets.
- **Linear MCP:** `search_tool` then `use_tool`. Schemas first. Inventory =
  eligibility **Linear inventory fetch** (`team` + `project` + `state`, slim
  fields). Never list the project unfiltered (dumps Done; truncates).

## Trigger phrases

`/stat`, `stat`, `board status`, `project status`, `what's open`,
`show issues that need to be resolved`, `what's left to solve`,
`open tickets urgent first`

## Invocation

```text
/stat
/stat 10
/stat checkout
/stat 5 admin
```

| Arg | Meaning |
|-----|---------|
| (none) | Every open issue, urgent → least |
| `N` (positive integer) | Show the top **N** after sort; still report full counts |
| other text | `AREA_FILTER` on title, labels, identifier (case-insensitive) |

Parse: bare positive integer → `TOP_N`; remainder → `AREA_FILTER`. Mention
ignored extra flags once.

---

## Skill paths

```text
STATUS_SKILL_DIR = directory containing this SKILL.md
```

Companions (first hit):

| Skill | Candidates |
|-------|------------|
| **issue** | `$STATUS_SKILL_DIR/../issue/SKILL.md` → `$HOME/.grok/skills/issue/SKILL.md` |
| **solve** | `$STATUS_SKILL_DIR/../solve/SKILL.md` → `$HOME/.grok/skills/solve/SKILL.md` |
| **identify ranking** | `$STATUS_SKILL_DIR/../identify/references/ranking.md` → `$HOME/.grok/skills/identify/references/ranking.md` |

Set `ISSUE_SKILL_MD`, `SOLVE_SKILL_DIR`, `RANKING_MD`.
`ELIGIBILITY_MD` = `$SOLVE_SKILL_DIR/references/eligibility.md`
(fallback `$HOME/.grok/skills/solve/references/eligibility.md`).

If **issue** is missing, still list teams/projects from Linear and ask once.
If **ranking** is missing, sort Linear priority `1, 2, 3|0, 4` then identifier;
skip `U`.

---

## Phase 0 — Bootstrap

1. Confirm workspace (prefer git). Read-only on app code. Do not discard
   dirty files. Do not fetch/merge/checkout.
2. Read `README.md` and applicable `AGENTS.md` / `CLAUDE.md`.
3. Parse args → `TOP_N` (or all), `AREA_FILTER`.

---

## Phase 1 — Resolve Linear team and project

Same automatic order as `/issue` Phase 1. Read `$ISSUE_SKILL_MD` Phase 1 and
follow it. Do not ask first. Ask **once** with top candidates only if
unresolved.

---

## Phase 2 — Inventory

Read `ELIGIBILITY_MD` **Linear inventory fetch** and follow it:

1. `list_issue_statuses`, then `list_issues` per kept status **name** with
   `team` + `project` + `state`, slim fields, page each state. Types
   `backlog` / `unstarted` / `started` only.
2. **Open set** = that union. Do **not** pull `description` on this pass.
3. `list_comments` **only** on `started` issues (In Progress / In Review /
   Blocked equivalents) to detect live `claimed-by:` (< 60 min).
4. Do **not** apply 2B as a hide filter. 2D/2E are wait-reasons, not drops.

If `AREA_FILTER` is set: keep issues whose title, labels, or identifier match.
If nothing matches: **stop** and ask whether to re-run unfiltered.

If the open set is empty: report team/project and **stop**. Do not invent work.

### Wait reason (one per issue)

Compute from inventory + started comments only:

| Signal | Wait |
|--------|------|
| Status/label Blocked, or open `blockedBy` if already loaded | `blocked` |
| Live foreign `claimed-by:` | `claimed` |
| In Progress, other assignee | `other-assignee` |
| In Review | `in-review` |
| Has children in this open set (`parentId` of others → this id) | `epic` |
| Child (`parentId` set) | `child of TEAM-…` |
| Else unstarted / unclaimed In Progress | `ready` |

Do not `get_issue` the whole board. Load relations only when a Blocked issue
has no wait reason yet.

---

## Phase 3 — Rank

Read `RANKING_MD` (Sort keys + Priority + User-facing impact). **Do not**
apply Size cut, Overlap cut, or Identify batch guidance.

1. Map Linear priority → `P1`–`P4` (`1` Urgent, `2` High, `3`/`0` Medium,
   `4` Low). Missing priority = Medium.
2. Score `U` from **title + labels** (no live app walk; no bulk
   descriptions). When unsure between adjacent scores, pick the **lower**.
3. Sort: `P` ascending, then `U` descending, then `TEAM-(\d+)` ascending.
4. If `TOP_N` is set, `SHOWN` = first `N`; else `SHOWN` = full ranked list.

---

## Phase 4 — Report, then stop

```markdown
**Status:** [Team] / [Project] · [open] open
**Rank:** Linear priority, then user-facing impact, then identifier
**Args:** top=[N or all] · area=[filter or none]
**Counts:** P1=n · P2=n · P3=n · P4=n · ready=n · blocked=n · claimed=n · in-progress=n · in-review=n · epics=n

### Queue (urgent → least)
1. [TEAM-123](url) — <title>
   P2 · U4 · Todo · unassigned · ready
2. [TEAM-80](url) — <title>
   P2 · U3 · In Progress · alice · claimed
3. [TEAM-40](url) — <title>
   P3 · U1 · In Review · me · in-review
```

Rules:

- Counts are for the **full open set** (after area filter), not only `SHOWN`.
- If `TOP_N` truncated: first line of the queue section =
  `Showing N of M`.
- One line of metadata per issue: `P · U · <state> · <assignee or unassigned> · <wait>`.
- Do not dump descriptions, code maps, or comments.
- Do not ask to approve a batch. Point at `/identify` only if they ask what
  to work next; `/prb` only for in-review rows if they ask how to ship.
- Then **stop**.

If Linear auth/tools fail: say which server failed. Do not invent issues.

---

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/stat` | Read-only **full open board**, urgent → least |
| `/identify` | Picks a **small** eligible batch and waits for approve |
| `/solve` | Implements the next eligible leaf |
| `/tidy` | Hygiene writes; not a briefing |
| `/issue` | Files one ticket; team/project resolution reused here |
| `/prb` | Ships `dev` → `main`; In Review rows are waiting on this |

```text
/stat              ← you are here (look)
        ↓
     /identify     → approve → /solve
        ↓
      /prb
```

---

## Anti-patterns

- Treating this as `/identify` (batch of 2–4, approve prompt, upgrades)
- Hiding blocked / claimed / In Review / epics because they are not
  `/solve`-eligible
- Sorting by identifier or “interesting” instead of P then U
- Applying Identify size/overlap cut
- `list_issues` for team/project with no `state`
- Pulling `description` for every issue
- Commenting, claiming, assigning, or changing state
- Starting `/solve` because the top row is “obvious”
- Listing Done / Canceled from an unfiltered project dump
- Using this skill for session `/status` (model, auth, context)

## Red flags — stop

- About to implement or claim
- About to show only eligible leaves and call that “the board”
- About to stop after the first page of one status
- About to ask “approve this batch?”
