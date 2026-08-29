---
name: identify
description: >
  Use when the user runs /identify, says "identify work", "pick a batch",
  "what should we solve", "recommend issues to solve", "choose Linear tickets
  to work", or wants a human-approved set of Linear tickets before /solve.
  Also /identify N, /identify TEAM-123, area/theme words, --pick-only, or fast.
argument-hint: "[N] [TEAM-123] [--pick-only] [fast] [area…]"
---

# /identify — Recommend a batch, then `/solve` on approve

Look at **open Linear issues for the current repo’s project**. Pick a **small
highest-value mix** of eligible leaves. Upgrade thin members of that set to the
`/issue` bar. **Stop and wait.**

This skill does **not** implement application code. Implementation happens only
via nested `/solve` after approve (unless `--pick-only`).

## Operating contract

- **Highest-value mix**, not a related cluster, unless the user passed an area
  filter. Rank = guidance bands (when a platform conflict exists) then Linear
  **priority**, then **user-facing impact**. Identifier number is a tie-break.
  [`references/ranking.md`](references/ranking.md).
- **Eligibility** = `$SOLVE_SKILL_DIR/references/eligibility.md` (2B / 2D / 2E).
  Expand epics; never recommend a parent shell. Do not invent a second
  blocked/claim policy.
- **Thin S0** before rank: [`references/guidance.md`](references/guidance.md)
  (uses `/solve` batch-guidance vocabulary).
- **Read-only on Linear** except: (1) **upgrade** bodies of the *proposed*
  batch before the approve prompt, (2) **JIT claim** of the next id after
  approve (not pick-only). No comments on ordinary skips.
- **Upgrade before the approve prompt.** [`references/upgrade.md`](references/upgrade.md).
- **First present = one batch.** Reply parsing (subset / drop / swap / theme)
  is [`references/replies.md`](references/replies.md).
- **Hard cap 4.** Default cut prefers **2**. Do not pad with Low/`U=0` chores.
- **Secrets:** never put tokens, env values, connection strings, or Doppler
  secrets in Linear or chat.
- **Linear MCP:** `search_tool` then `use_tool`. Schemas first. Literal
  newlines in markdown bodies. **Before every `save_comment`:** `list_comments`
  on that issue ([`../docs/linear-comments.md`](../docs/linear-comments.md)).
  Inventory = eligibility **Linear inventory fetch** (`team` + `project` +
  `state`, slim fields). Never dump the whole project including Done.

## Trigger phrases

`/identify`, `identify work`, `pick a batch`, `what should we solve`,
`recommend issues to solve`, `choose Linear tickets to work`,
`what’s the best batch to solve`

## Invocation

```text
/identify
/identify 2
/identify 3
/identify TEAM-123
/identify 2 TEAM-123
/identify checkout
/identify admin apps/web
/identify --pick-only
/identify pick-only
/identify fast
/identify 2 checkout --pick-only
```

| Arg | Meaning |
|-----|---------|
| (none) | Rank + cut; `MAX_N = 4` cap; default cut prefers 2 |
| `N` (1–4) | `MAX_N = N` |
| `TEAM-123` | `PIN_ID` — first slot if eligible; fill remaining by rank |
| `--pick-only` / `pick-only` | On approve: print ids; **no** claim; **no** `/solve` |
| `fast` / `--fast` | `FAST_ON_APPROVE`. After approve, parallel `/solve` **only** if `FAST_OK` |
| other text | `AREA_FILTER` on title, labels, description, code-map paths |

### Parse order

1. `--pick-only` / `pick-only` → `PICK_ONLY`
2. `fast` / `--fast` → `FAST_ON_APPROVE`
3. `TEAM-\d+` → `PIN_ID`
4. Bare integer **1–4** → `MAX_N` (else `MAX_N = 4`)
5. Remainder → `AREA_FILTER` (mention once if present)

---

## Skill paths

```text
IDENTIFY_SKILL_DIR = directory containing this SKILL.md
RANKING_MD         = $IDENTIFY_SKILL_DIR/references/ranking.md
UPGRADE_MD         = $IDENTIFY_SKILL_DIR/references/upgrade.md
GUIDANCE_MD        = $IDENTIFY_SKILL_DIR/references/guidance.md
REPLIES_MD         = $IDENTIFY_SKILL_DIR/references/replies.md
```

Resolve companions (first hit):

| Skill | Candidates |
|-------|------------|
| **issue** | `$IDENTIFY_SKILL_DIR/../issue/SKILL.md` → `$HOME/.grok/skills/issue/SKILL.md` |
| **solve** | `$IDENTIFY_SKILL_DIR/../solve/SKILL.md` → `$HOME/.grok/skills/solve/SKILL.md` |

Set `ISSUE_SKILL_MD`, `ISSUE_SKILL_DIR`, `SOLVE_SKILL_MD`, `SOLVE_SKILL_DIR`.
`ELIGIBILITY_MD` = `$SOLVE_SKILL_DIR/references/eligibility.md`.
`BATCH_GUIDANCE_MD` = `$SOLVE_SKILL_DIR/references/batch-guidance.md`.
`MULTIPLAYER_MD` = `$SOLVE_SKILL_DIR/references/multiplayer-linear.md`.

If **solve** is missing, stop after a recommendation — do not pretend `/solve`
ran. If **issue** is missing, still recommend; skip Linear upgrades and mark
each thin ticket as not upgraded.

---

## Phase 0 — Bootstrap

1. Confirm workspace root (git repo when possible). Read-only on app code until
   nested `/solve` starts. Do not discard unrelated dirty files.
2. Read root `README.md` and applicable `AGENTS.md` / `CLAUDE.md`.
3. Parse invocation (table above).
4. Initialize:
   - `RUN_ID` = short unique id (reuse for claims + nested solve)
   - `REJECTED = []`
   - `PROPOSED = []`
   - `UPGRADED = []`
   - `FAST_OK = false`
5. Read `RANKING_MD`, `UPGRADE_MD`, `GUIDANCE_MD`, `REPLIES_MD` now.

---

## Phase 1 — Resolve Linear team and project

Same automatic order as `/issue` Phase 1. **Read** `$ISSUE_SKILL_MD` Phase 1
and follow it. Do not ask first. Ask **once** with top candidates only if
unresolved.

Reuse the resolution for every reject/re-propose loop in this session.

---

## Phase 2 — Inventory and eligibility

1. Read `ELIGIBILITY_MD`. Fetch the board with **Linear inventory fetch**
   (`list_issue_statuses`, then `list_issues` per kept status name with
   `team` + `project` + `state`, slim fields, page each state). Do **not**
   list the project unfiltered. Do **not** pull `description` on this pass.
2. Apply 2B, 2D, 2E on that union. Identify inventory is **read-only** (no
   epic rollup writes). `get_issue` / `list_comments` only for 2D/2E needs
   and for tickets that enter rank / `PROPOSED`.
3. If `AREA_FILTER`: keep leaves whose title, labels, description, or code-map
   paths match (case-insensitive). If the filter matches nothing: **stop** and
   ask whether to re-run unfiltered. Do not silently drop the filter.
4. Result = `ELIGIBLE`: implementable **leaves** only, not in `REJECTED`.
5. Record skip reasons internally. **Do not** comment on those issues.

If `PIN_ID` is set and that issue is ineligible: say why once; continue without
it.

If `ELIGIBLE` is empty: report team/project, rejected ids (if any), and why
nothing is implementable. **Stop.** Do not invent work.

---

## Phase 2.5 — Thin S0

Follow [`references/guidance.md`](references/guidance.md). Uses
`BATCH_GUIDANCE_MD` for tags/conflicts. Drops full-obsolete from `ELIGIBLE`.
Promotes canonical migrations **only when a platform conflict exists**.

If direction confidence is **low**: stop and ask. Do not rank yet.

---

## Phase 3 — Rank, cut, overlap

Follow [`references/ranking.md`](references/ranking.md):

1. Score remaining `ELIGIBLE` (priority, then U).
2. Sort (guidance band if conflict, then P, then U, then number).
3. Cut using `MAX_N` and the size rule (prefer 2).
4. Overlap cut. Set `FAST_OK`.
5. `PROPOSED` = that ordered list.

---

## Phase 4 — Upgrade thin tickets in `PROPOSED`

Follow [`references/upgrade.md`](references/upgrade.md). Status line first,
then parallel research / serial Linear save. Tidy-stamp skip when fresh and
still ready.

If an upgrade cannot reach the bar: drop that id, pull the next ranked eligible
leaf, upgrade that one. Repeat until the batch is ready or `ELIGIBLE` is
exhausted.

---

## Phase 5 — Present and wait

Show **one** batch. Then **stop**. Do not claim. Do not start `/solve`.

```markdown
**Identify:** [N] issues · [Team] / [Project] · run <RUN_ID>
**Rank:** [guidance bands ·] Linear priority, then user-facing impact
**Args:** MAX_N=[n] · pin=[id or none] · area=[filter or none] · pick-only=[yes/no] · fast=[ok/refused/off]

### Proposed
1. [TEAM-123](url) — <title>
   **Why:** P[priority] · U[n] · <impact one-liner>
   **Ready:** upgraded | already ready | already ready (tidy)
2. …

### Not in this batch
- Rejected this session: TEAM-… (omit if none)
- Eligible but lower rank: TEAM-… — <priority + impact> (cap at ~5)
- Ineligible / obsolete-skip: TEAM-… — blocked / claimed / other assignee / epic / abandoned stack
  (only mention high-priority misses; do not dump the whole board)

Approve to [claim+`/solve 1` each | print ids only if pick-only], in order.
Reject for the next-best set (these IDs excluded).
Subset / drop / swap / theme also work.
```

Then wait. Parse the reply with [`references/replies.md`](references/replies.md).

---

## Phase 6 — Reject / rebuild

On **whole-batch reject:** append every `PROPOSED` id to `REJECTED`, clear
`PROPOSED` / this-round `UPGRADED` display list (Linear upgrades stay), re-run
Phase 3–5 on `ELIGIBLE` minus `REJECTED`. Refresh Linear states if the board
may have changed. Do not re-upgrade tickets already upgraded this session
unless someone else edited them.

On **swap / theme:** follow replies.md (re-present once for swap).

If nothing remains: eligible board is exhausted for this session. **Stop.**

---

## Phase 7 — Approve → JIT claim → nested `/solve`

`QUEUE = PROPOSED` in presented order (after subset/drop).

### 7A. `--pick-only`

Print the ids + URLs + order. **Stop.** No claim. No `/solve`.

### 7B. Claim just-in-time

Read `MULTIPLAYER_MD`. **Do not** claim the whole queue up front.

For each `ID` in `QUEUE`, **immediately before** its nested `/solve`:

1. Confirm still unclaimed (or already claimed by this `RUN_ID`).
   **`list_comments` first.** Skip a new claim comment if this `RUN_ID` already
   has a live `claimed-by:` on the issue.
2. Assign to me if unassigned. Set **In Progress**.
3. Comment, first line exactly:

```text
claimed-by: identify · session <session> · worktree <cwd> · run <RUN_ID>
```

Then one short paragraph: identify-approved queue (sibling ids + order);
this leaf is next; nested `/solve 1` will implement.

4. Re-fetch. If a **different** run’s `claimed-by:` is newer: drop that leaf
   from `QUEUE`, tell the user, continue with the rest.

Do **not** claim parent epics. Do **not** mark Done.

If the **current** claim loses CAS: skip that id, do not claim ahead. If
every remaining id loses CAS: stop and report. Do not start `/solve`.

### 7C. Nested `/solve` — subagent, one ID at a time

Identify owns the queue. `/solve` only pins the first `TEAM-123` token.

**Sequential (default, and whenever `FAST_OK` is false):**

```text
for ID in QUEUE:
  JIT-claim ID (7B)
  spawn_subagent general-purpose (nested /solve orchestrator), isolation: "none", model: "grok-4.6"
  (see [`../docs/grok-models.md`](../docs/grok-models.md); do not inherit parent)
  prompt must include:
    - Read $SOLVE_SKILL_MD and follow it end-to-end (Phase 0–9)
    - SOLVE_COUNT_MODE = 1
    - preferred issue = ID
    - SELECTION_PIN = [ID]
    - FAST_MODE = false
    - RUN_ID = this identify RUN_ID
    - claimed-by: identify with this RUN_ID is this run, not foreign
    - Do not pick any other leaf
    - Dirty tree: do not discard unrelated files
    - Same git contract as /solve (local dev, no push unless the user asked)
  Identify must not write application source while the worker runs
  if that solve fails (implement / verify / merge):
    stop the queue
    release every later QUEUE id that this RUN_ID claimed and did not start
      (should be none under JIT)
    current failed leaf stays In Progress (solve owns it)
    report what finished and what is still claimed
  else:
    continue to the next ID
```

If `FAST_ON_APPROVE` but not `FAST_OK`: warn once (overlap or platform
conflict), then sequential.

**Fast (`FAST_OK`):** one `general-purpose` subagent, `isolation: none`, `model: grok-4.6`:

```text
Read $SOLVE_SKILL_MD + fast-mode.md
FAST_MODE = true
SOLVE_COUNT_MODE = len(QUEUE)
SELECTION_PIN = QUEUE (ordered)
RUN_ID = this identify RUN_ID
GUIDANCE_REQUIRED = true
Do not refill outside SELECTION_PIN
Treat identify claimed-by with this RUN_ID as this run
```

Identify still must not write application source. On fast worker hard-fail:
release any QUEUE id this run claimed that was not started; leave in-flight
leaves as solve left them.

### 7D. Release

On queue stop, user abort, or pick-only after a mistaken claim, for each leaf
this `RUN_ID` claimed that is **not** solved and **not** the in-flight failure:

1. `list_comments` first. Skip if `released: identify-stopped · run <RUN_ID>`
   already exists. Else comment, first line: `released: identify-stopped · run <RUN_ID>`
2. Set Backlog/Todo (team default for unstarted)
3. Unassign if Identify assigned them this run

Do not release a leaf the nested solve already moved to In Review / completed.

### 7E. User reply after the queue

Reuse solve’s single-issue or multi-issue summary shape, plus:

```markdown
**Identify:** approved [id → id → …]
**Claimed run:** <RUN_ID>
**Mode:** sequential `/solve 1` | fast pin | pick-only
**Solved:** …
**Stopped early:** <ID + reason>
**Still claimed (not solved):** …
**Released:** …
```

Omit empty rows.

---

## Linear hygiene

| Moment | Linear write? |
|--------|----------------|
| Inventory / skip / thin S0 | No |
| Upgrade thin proposed ticket | **Yes** — body (and title only if vague) |
| Present batch | No |
| Reject | No |
| Approve pick-only | No |
| Approve + solve | **Yes** — JIT assign, In Progress, `claimed-by:` on the **next** id only |
| Nested `/solve` | Solve owns start-plan/closeout on that leaf |
| Queue stop (unstarted claims) | **Yes** — `released:` + Backlog/Todo |
| Parent epic | No claim; solve may comment if it expands |

---

## Relation to other skills

| Skill | Role |
|-------|------|
| `/identify` | Human-approved **batch picker** + JIT claim + start `/solve` |
| `/stat` | Read-only full open board, urgent → least; does not pick or claim |
| `/issue` | Team/project resolution + execution-ready bar (upgrade target) |
| `/solve` | Eligibility file, claim format, cheap construction, merge local `dev` |
| `/tidy` | Whole-board hygiene + `tidy-pass:` stamp Identify may skip on |
| `/project-review` | Creates the queue Identify reads (whole-project review) |
| `/walk` | Creates the queue from a live front-facing UI walk |
| `/prb` | Ships `dev` after solve; not used here |

```text
/project-review | /walk | /issue | /issues
        ↓
     /tidy         (optional weekly)
        ↓
   /identify     →  approve  →  /solve 1 per ID (or fast pin)
        ↓ reject
   next-best set
        ↓
      /prb         (later, not this skill)
```

---

## Anti-patterns

- Starting `/solve` before an explicit approve
- Calling `/solve N id1 id2` and assuming solve will honor the full ID list
  without `SELECTION_PIN`
- Ranking by U only across an abandoned vs canonical platform split
- Padding the batch with Low/docs tickets to reach 4
- Recommending an epic/parent shell
- Claiming the whole batch before the first `/solve`
- Leaving unsolved queue ids In Progress after a stop
- Claiming or commenting on issues that were only scanned
- Showing the approve prompt before upgrading thin members
- Creating new Linear issues instead of updating thin ones
- Marking Done from Identify
- Pushing or opening PRs (unless the user asked; `/solve` still defaults local)
- Inventing a second blocked/claim policy instead of `eligibility.md`
- Ignoring `REJECTED` and re-proposing the same ids
- Loading the entire `/solve` skill just to apply 2B–2E (read `eligibility.md`)
- Treating `claimed-by: identify` as foreign when `run` matches `RUN_ID`
- Nested `/solve` or upgrade workers without `model: grok-4.6` / `grok-4.5` ([`../docs/grok-models.md`](../docs/grok-models.md))
- Posting a claim/release comment without `list_comments` first
- `list_issues` for team/project with no `state` (dumps Done; truncates)

## Red flags — stop

- About to implement because the batch “is obvious”
- About to claim 2–4 tickets before any worker starts
- About to add a chore so the batch looks fuller
- About to pick the epic because it is High
- About to `/solve` a ClickHouse/Convex (etc.) feature ahead of an open
  canonical migration
- About to treat a non-yes reply as invalid instead of subset/drop/swap/theme

Pressure cases: [`evals/pressure-scenarios.md`](evals/pressure-scenarios.md).
