---
name: identify
description: >
  Inspect open Linear issues for the current repo’s project, pick a small
  high-value batch (2–4 unblocked leaves), upgrade thin tickets to the /issue
  execution-ready bar, then wait for the user to approve or reject. Approve
  claims the batch and runs sequential /solve 1 on each exact ID. Reject
  proposes the next-best 2–4 excluding rejected IDs. Rank by Linear priority
  then user-facing impact. Use when the user runs /identify, says "identify
  work", "pick a batch", "what should we solve", "recommend issues to solve",
  or wants a human-approved set of Linear tickets before /solve.
---

# /identify — Recommend a batch, then /solve on approve

Look at **open Linear issues for the current repo’s project**. Pick a **small
highest-value mix** (2–4 eligible leaves). Upgrade any thin tickets in that set
to the `/issue` bar **before** showing them. **Stop and wait.**

- **Approve** → claim every ID in the batch, then run sequential `/solve 1 <ID>`
  in the recommended order.
- **Reject** → exclude those IDs for this session and propose the next-best 2–4.

This skill does **not** implement application code itself. Implementation happens
only via nested `/solve` after approve.

## Operating contract

- **Plain invocation only.** `/identify` takes no size, theme, or issue-id args.
  Extra words after `/identify` are ignored (mention that once if present).
- **Default batch size: 2–4.** Do not pad with low-value chores to hit 4. Do not
  exceed 4. A board with one eligible leaf yields a batch of 1.
- **Highest-value mix**, not a related cluster. Unrelated tickets in one batch
  are expected when they outrank a theme.
- **Rank** = Linear **priority first**, then **user-facing impact**. Issue number
  is a tie-break only. Full rules: [`references/ranking.md`](references/ranking.md).
- **Eligibility** = `/solve` Phase 2B–2E (unblocked implementable **leaves**).
  Expand epics; never recommend a parent shell. Read the solve skill for the
  definition — do not invent a second blocked/claim policy.
- **Read-only on Linear** except: (1) **upgrade** bodies of the *proposed* batch
  before display, (2) **claim** on approve. No comments on ordinary skips.
- **Upgrade before display.** Rewrite thin proposed tickets in Linear first, then
  show the batch. Procedure: [`references/upgrade.md`](references/upgrade.md).
- **One batch + yes/no.** Do not offer a shortlist, edits, or alternative mixes
  on the first pass.
- **Secrets:** never put tokens, env values, connection strings, or Doppler
  secrets in Linear or chat.
- **Linear MCP:** `search_tool` then `use_tool`. Read schemas first. Literal
  newlines in markdown bodies.

## Trigger phrases

`/identify`, `identify work`, `pick a batch`, `what should we solve`,
`recommend issues to solve`, `choose Linear tickets to work`,
`what’s the best batch to solve`

---

## Skill paths

```text
IDENTIFY_SKILL_DIR = directory containing this SKILL.md
RANKING_MD         = $IDENTIFY_SKILL_DIR/references/ranking.md
UPGRADE_MD         = $IDENTIFY_SKILL_DIR/references/upgrade.md
```

Resolve companions (first hit):

| Skill | Candidates |
|-------|------------|
| **issue** | `$IDENTIFY_SKILL_DIR/../issue/SKILL.md` → `$HOME/.grok/skills/issue/SKILL.md` |
| **solve** | `$IDENTIFY_SKILL_DIR/../solve/SKILL.md` → `$HOME/.grok/skills/solve/SKILL.md` |

Set `ISSUE_SKILL_MD`, `ISSUE_SKILL_DIR`, `SOLVE_SKILL_MD`, `SOLVE_SKILL_DIR`.
If **solve** is missing, stop after a recommendation — do not pretend `/solve`
ran. If **issue** is missing, still recommend; skip Linear upgrades and mark
each thin ticket in the batch as not upgraded.

---

## Phase 0 — Bootstrap

1. Confirm workspace root (git repo when possible). Read-only on app code until
   nested `/solve` starts. Do not discard unrelated dirty files.
2. Read root `README.md` and applicable `AGENTS.md` / `CLAUDE.md`.
3. Parse invocation: **ignore** size, `fast`, issue ids, and theme words.
4. Initialize:
   - `RUN_ID` = short unique id for this identify session (reuse for claims + nested solve)
   - `REJECTED = []` (identifiers the user rejected this session)
   - `PROPOSED = []`
   - `UPGRADED = []`
5. Read `RANKING_MD` and `UPGRADE_MD` now.

---

## Phase 1 — Resolve Linear team and project

Same automatic order as `/issue` Phase 1. **Read** `$ISSUE_SKILL_MD` Phase 1
(or the issue skill’s “Resolve Linear team and project” section) and follow it.
Do not ask first. Ask **once** with top candidates only if unresolved.

Reuse the resolution for every reject/re-propose loop in this session.

---

## Phase 2 — Inventory and eligibility

1. `list_issues` for the resolved team/project. Page until complete.
2. Load `$SOLVE_SKILL_MD` and apply **Phase 2B–2E** (eligible states, sort is
   **not** used yet, blocked definition, epic/parent expansion).
3. Result = `ELIGIBLE`: implementable **leaves** only, not in `REJECTED`.
4. Record skip reasons internally (blocked, other-assignee, live foreign
   `claimed-by:`, Done/Canceled, epic with no eligible child). **Do not**
   comment on those issues.

If `ELIGIBLE` is empty: report team/project, rejected ids (if any), and why
nothing is implementable. **Stop.** Do not invent work.

---

## Phase 3 — Rank and cut a batch

Follow [`references/ranking.md`](references/ranking.md):

1. Score every `ELIGIBLE` leaf (priority, then user-facing impact).
2. Sort: priority (Urgent → Low), then impact (high → low), then identifier
   number ascending.
3. Cut **2–4** from the top using the size rule in ranking.md.
4. Set `PROPOSED` to that ordered list.

---

## Phase 4 — Upgrade thin tickets in `PROPOSED`

For each id in `PROPOSED`, follow [`references/upgrade.md`](references/upgrade.md):

- Already execution-ready → leave the Linear body alone.
- Thin → investigate the repo, rewrite the **existing** issue to the `/issue`
  bar, save in Linear (update, do not create a duplicate). Append to `UPGRADED`.

Do this **before** Phase 5. The user must approve a ready batch.

If an upgrade cannot reach the bar (missing product intent, no code pin):
drop that id from `PROPOSED`, treat it as skipped this round, and pull the next
ranked eligible leaf into the batch (then upgrade that one). Repeat until the
batch is 2–4 ready leaves or `ELIGIBLE` is exhausted.

---

## Phase 5 — Present and wait

Show **one** batch. Then **stop**. Do not claim. Do not start `/solve`.

```markdown
**Identify:** [N] issues · [Team] / [Project]
**Rank:** Linear priority, then user-facing impact

### Proposed
1. [TEAM-123](url) — <title>
   **Why:** P[priority] · <impact one-liner>
   **Ready:** upgraded | already ready
2. …

### Not in this batch
- Rejected this session: TEAM-… (omit if none)
- Eligible but lower rank: TEAM-… — <priority + impact> (cap at ~5)
- Ineligible: TEAM-… — blocked / claimed / other assignee / epic not expandable
  (only mention high-priority misses; do not dump the whole board)

Approve to claim these and run `/solve 1` on each, in order.
Reject to see the next-best 2–4 (these IDs excluded).
```

Approve: `yes`, `approve`, `go`, `lgtm`, `do it`, or equivalent clear yes.  
Reject: `no`, `reject`, `skip`, `next`, `different`, or equivalent clear no.

If the user writes something that is neither: ask once, yes or no.

---

## Phase 6 — Reject → next batch

On reject:

1. Append every id in `PROPOSED` to `REJECTED`.
2. Clear `PROPOSED` / this-round `UPGRADED` display list (Linear upgrades stay).
3. Re-run Phase 3–5 on `ELIGIBLE` minus `REJECTED` (refresh Linear states if
   the board may have changed; do not re-upgrade tickets already upgraded
   this session unless they were edited by someone else).

If nothing remains: say the eligible board is exhausted for this session and
**stop**.

---

## Phase 7 — Approve → claim → sequential `/solve`

### 7A. Claim the whole batch

Read `$SOLVE_SKILL_DIR/references/multiplayer-linear.md` and claim **each**
`PROPOSED` leaf **before** any git/implement work:

1. Confirm still unclaimed (or already claimed by this `RUN_ID`).
2. Assign to me if unassigned. Set **In Progress**.
3. Comment, first line exactly:

```text
claimed-by: identify · session <session> · worktree <cwd> · run <RUN_ID>
```

Then one short paragraph: identify-approved batch (list sibling ids + order);
nested `/solve 1` will implement.

4. Re-fetch. If a **different** run’s `claimed-by:` is newer: drop that leaf
   from the solve queue, tell the user, continue with the rest.

Do **not** claim parent epics. Do **not** mark Done.

If **every** claim loses the CAS check: stop and report. Do not start `/solve`.

### 7B. Nested `/solve` — one ID at a time

`/solve` only pins the **first** `TEAM-123` token. Do **not** call
`/solve N id1 id2 id3` and expect a queue. Identify owns the queue:

```text
QUEUE = PROPOSED in presented order, minus CAS losses
for ID in QUEUE:
  run /solve as a nested procedure with:
    SOLVE_COUNT_MODE = 1
    preferred issue = ID
    FAST_MODE = false
    RUN_ID = this identify RUN_ID   # treat identify claims as this run
  if that solve fails (implement / verify / merge):
    stop the queue
    leave remaining QUEUE ids In Progress + claimed
    report what finished and what is still claimed
  else:
    continue to the next ID
```

Load `$SOLVE_SKILL_MD` and follow it end-to-end for each ID (Phase 0–9 of
solve, with count mode `1`). If nested `/solve` is a `spawn_subagent`, pass
**`model: grok-4.6`** ([`../docs/grok-models.md`](../docs/grok-models.md)).
Identify must **not** write application source during those loops; the solve
orchestrator / implementers do.

Hard constraints to inject into each nested solve:

- This leaf is already claimed by `RUN_ID` — do not treat the identify
  `claimed-by:` as foreign; do not abort on it.
- Implement **only** this `ID` (already a `/solve 1`).
- Sequential only. No `fast`.
- Same git contract as `/solve` (local `dev`, no push unless the user asked
  in this session).

### 7C. User reply after the queue

Reuse solve’s single-issue or multi-issue summary shape, plus:

```markdown
**Identify:** approved [id → id → …]
**Claimed run:** <RUN_ID>
**Solved:** …
**Stopped early:** <ID + reason>   <!-- omit if the queue finished -->
**Still claimed (not solved):** …  <!-- omit if none -->
```

---

## Linear hygiene

| Moment | Linear write? |
|--------|----------------|
| Inventory / skip | No |
| Upgrade thin proposed ticket | **Yes** — body (and title only if vague) |
| Present batch | No |
| Reject | No |
| Approve | **Yes** — assign, In Progress, `claimed-by:` |
| Nested `/solve` | Solve owns start-plan/closeout on that leaf |
| Parent epic | No claim; solve may comment if it expands |

---

## Relation to other skills

| Skill | Role |
|-------|------|
| `/identify` | Human-approved **batch picker** + claim + start `/solve` |
| `/issue` | Team/project resolution + execution-ready bar (upgrade target) |
| `/solve` | Eligibility, claim format, implement→review, merge local `dev` |
| `/project-review` | Creates the queue Identify reads |
| `/prb` | Ships `dev` after solve; not used here |

```text
/project-review | /issue | /issues
        ↓
   /identify     →  approve  →  /solve 1 per ID (sequential)
        ↓ reject
   next-best 2–4
        ↓
      /prb       (later, not this skill)
```

---

## Anti-patterns

- Starting `/solve` before an explicit approve
- Calling `/solve N id1 id2` and assuming solve will honor the full ID list
- Picking by theme, lowest number, or unblock-count instead of priority + impact
- Padding the batch with Low/docs tickets to reach 4
- Recommending an epic/parent shell
- Claiming or commenting on issues that were only scanned
- Showing the batch before upgrading thin members
- Creating new Linear issues instead of updating thin ones
- Marking Done from Identify
- Pushing or opening PRs (unless the user asked; `/solve` still defaults local)
- Inventing a second blocked/claim policy instead of reading `/solve`
- Ignoring `REJECTED` and re-proposing the same ids
