---
name: tidy
description: >
  Hygiene pass on the current repo’s Linear project: upgrade thin issues to the
  /issue bar, retitle and fix relations, roll up finished epics, mark
  high-confidence duplicates/obsolete tickets, and set status when work is
  already shipped (In Review if on origin/dev, Done only if on main). Skip any
  issue tidied in the last 7 days unless /tidy --force or /tidy TEAM-123.
  Finish the board first, then one needs-you list. Skip live foreign claims.
  Track last pass via a Linear tidy-pass stamp plus a local ledger. Use when
  the user runs /tidy, /tidy --force, /tidy TEAM-123, says "tidy Linear",
  "clean up the board", "thicken thin tickets", or "close issues that are
  already done".
argument-hint: "[--force] [TEAM-123]"
---

# /tidy — Linear board hygiene for the current project

Inspect **this repo’s Linear project**. For every **due** issue: thicken thin
tickets, fix obvious titles/relations, close or duplicate high-confidence
dead wood, roll up finished epics, and move status when the work is **already
shipped**. Then **stop** with one needs-you list.

This skill does **not** implement application code, claim work for `/solve`,
push, or open PRs.

## Operating contract

- **Whole due board** by default. `/tidy TEAM-123` is that issue only (cooldown
  ignored). `/tidy --force` ignores the 7-day cooldown for the whole board.
- **Cooldown: 7 days** per issue unless overridden. Ledger + stamp:
  [`references/ledger.md`](references/ledger.md).
- **Quality bar** = `/issue` (same as Identify upgrades). Do not invent a
  second template.
- **Status** matches `/solve` + `/prb`: **In Review** if shipped to
  `origin/dev` and not on `main`; **Done** only if merged to `origin/main`
  (or the team’s trunk). Never Done from **local-only** work.
- **High-confidence writes apply immediately**, including Cancel/Duplicate.
  Low-confidence closes are listed, not applied. Rules:
  [`references/actions.md`](references/actions.md).
- **Live foreign claim** → skip entirely. No rewrite, no status, no stamp,
  no cooldown. See `/solve` `references/multiplayer-linear.md`.
- **Needs-you last.** Do not stop the pass. Do not set Blocked. Collect
  questions; ask once at the end.
- **No implement, no assign-for-solve, no `claimed-by:`.**
- **Secrets:** env **names** only.
- **Linear MCP:** `search_tool` then `use_tool`. Schemas first. Literal
  newlines in markdown bodies.

## Trigger phrases

`/tidy`, `/tidy --force`, `/tidy TEAM-123`, `tidy Linear`, `tidy the board`,
`clean up Linear issues`, `thicken thin tickets`, `close issues that are
already done`

## Invocation

```text
/tidy
/tidy --force
/tidy force
/tidy TEAM-123
/tidy --force TEAM-123
```

| Arg | Meaning |
|-----|---------|
| (none) | Every **due** issue on the resolved project (last pass ≥ 7 days ago or never) |
| `--force` / `force` | Ignore cooldown; still skip live foreign claims |
| `TEAM-123` | That issue only; ignore its cooldown. Do not scan the rest of the board for writes |

Parse: extract `--force`/`force`, then a `TEAM-\d+` token. Ignore other words
(mention once).

---

## Skill paths

```text
TIDY_SKILL_DIR = directory containing this SKILL.md
LEDGER_MD      = $TIDY_SKILL_DIR/references/ledger.md
ACTIONS_MD     = $TIDY_SKILL_DIR/references/actions.md
```

Companions (first hit):

| Skill | Candidates |
|-------|------------|
| **issue** | `$TIDY_SKILL_DIR/../issue/SKILL.md` → `$HOME/.grok/skills/issue/SKILL.md` |
| **solve** | `$TIDY_SKILL_DIR/../solve/SKILL.md` → `$HOME/.grok/skills/solve/SKILL.md` |
| **identify upgrade** (optional) | `$TIDY_SKILL_DIR/../identify/references/upgrade.md` |

If **issue** is missing, still do status/dup/rollup; skip body upgrades and
mark those as not upgraded. If **solve** is missing, still skip anything with
a live `claimed-by:` comment younger than 60 minutes.

---

## Phase 0 — Bootstrap

1. Confirm workspace (prefer git). Read-only on app code. Do not discard
   dirty files. `git fetch origin` when remotes exist (needed for ship
   evidence). Do not merge or checkout away from the user’s branch.
2. Read `README.md` and applicable `AGENTS.md` / `CLAUDE.md`.
3. Parse args → `FORCE`, `PINNED_ID`.
4. `RUN_ID` = short unique id. `NEEDS_YOU = []`. `SKIPPED = []`.
   `CHANGED = []`. `INSPECTED_OK = []`.
5. Read `LEDGER_MD` and `ACTIONS_MD`.

---

## Phase 1 — Resolve Linear team and project

Same automatic order as `/issue` Phase 1. Read `$ISSUE_SKILL_MD` and follow
it. Ask **once** only if unresolved.

---

## Phase 2 — Inventory

1. `list_issues` for the team/project. Page until complete.
2. Include open/actionable states **and** In Review / Blocked / In Progress.
   Terminal Done/Canceled/Duplicate are **not** tidy targets (except as
   evidence that an epic’s children are terminal).
3. Load comments on candidates that might have `tidy-pass:` or `claimed-by:`.
   Prefer listing comments only when the local ledger has no fresh stamp
   **or** you need claim detection (In Progress).
4. Load the local ledger ([`references/ledger.md`](references/ledger.md)).
   **Linear `tidy-pass:` comment wins** when both exist.

### Due vs skip

| Condition | Action |
|-----------|--------|
| `PINNED_ID` set and this is not that issue | Ignore (out of scope) |
| Live foreign `claimed-by:` (< 60 min, different run) or In Progress assigned to someone else | **Skip claimed** — no writes, no stamp |
| Last `tidy-pass` < 7 days and not `FORCE` and not pinned | **Skip cooldown** |
| Else | **Due** |

If `PINNED_ID` is set, the due set is that one issue (unless claimed-skip).

If nothing is due: report counts (cooldown / claimed / terminal) and **stop**.

---

## Phase 3 — Classify and act each due issue

For each due issue (lowest identifier number first; parents after their
children when both are due):

Follow [`references/actions.md`](references/actions.md) in this order:

1. **Claimed?** already filtered.
2. **Completed in git?** → In Review or Done + evidence comment.
3. **Epic rollup?** all children terminal → Done + rollup comment.
4. **High-confidence duplicate / fully obsolete?** → Duplicate or Canceled +
   one comment (canonical or superseding id).
5. **Title** vague → retitle.
6. **Relations** obvious (`relatedTo` / `blockedBy` / parent) → set.
7. **Thin body?** investigate like `/issue` Phase 3; update the **existing**
   issue to the `/issue` bar. Ready → leave body alone.
8. **Needs a human?** do not guess. Append to `NEEDS_YOU`. Leave state.
   Still stamp the pass so we do not re-ask for a week.

Apply high-confidence Linear writes as you go. Do not implement code.

If upgrade cannot reach the `/issue` bar without a product decision or a
code pin, do **not** save a half body. Record `needs-you` instead.

Stamp every due issue you actually processed (including inspect-only and
needs-you). Do **not** stamp claimed-skips or cooldown-skips.

---

## Phase 4 — Ledger

After the due set is processed, write:

1. A `tidy-pass:` comment on each processed issue (see ledger.md).
2. The local ledger file (create parent dirs). Merge with previous entries.

---

## Phase 5 — Report, then needs-you

```markdown
**Tidy:** [Team] / [Project] · run <RUN_ID>
**Scope:** all due | force | pinned TEAM-123
**Due / cooldown-skip / claimed-skip:** D / C / K

### Changed
- [TEAM-123](url) — upgraded · retitled · relatedTo TEAM-80
- [TEAM-124](url) — status In Review (`origin/dev` <sha>)
- [TEAM-125](url) — Duplicate of TEAM-90
- [TEAM-100](url) — epic rollup Done

### Inspected, already tidy
- TEAM-… (ready, status correct)

### Skipped
- Cooldown (< 7d): TEAM-…
- Live claim: TEAM-…

### Needs you
1. [TEAM-130](url) — <one precise question>
2. …
```

If `NEEDS_YOU` is empty, omit that section and stop.

If `NEEDS_YOU` is non-empty: **stop and wait**. Do not start `/identify` or
`/solve`. When the user answers in a follow-up, apply answers to those
Linear bodies/assumptions (and status only if they explicitly confirm a
close). Do **not** re-scan the whole board unless they run `/tidy` again.

---

## Linear hygiene

| Write | When |
|-------|------|
| Body / title | Thin or vague, after research |
| `relatedTo` / `blockedBy` / parent | High-confidence only |
| In Review | Evidence on `origin/dev`, not on `main` |
| Done | Evidence on `origin/main`, or epic all children terminal |
| Duplicate / Canceled | High-confidence only + one comment |
| `tidy-pass:` comment | Every processed due issue |
| Assignee / `claimed-by:` / In Progress | **Never** |
| App source / git branch / push | **Never** |

---

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/tidy` | Board hygiene + stamps. No implement |
| `/issue` | Quality bar + team/project resolution |
| `/identify` | Picks 2–4 to **solve**; upgrades only that batch; waits to approve implement |
| `/solve` | Implements; claim protocol Tidy must not fight |
| `/prb` | Done after merge to `main` — Tidy may set Done only with the same evidence |

```text
/project-review | /walk | /issue | /issues
        ↓
     /tidy      →  board ready (weekly)
        ↓
   /identify    →  approve  →  /solve
        ↓
      /prb
```

---

## Anti-patterns

- Marking **Done** from local `dev` only
- Rewriting or closing a **live foreign claim**
- Re-tidying an issue inside 7 days without `--force` or a pinned id
- Stopping the pass to ask the first needs-you question
- Setting Blocked for needs-you
- Implementing the ticket
- Creating a new issue instead of upgrading the thin one
- Canceling on low-confidence “maybe obsolete”
- Stamping cooldown on an issue you skipped
- Inventing a second quality template instead of reading `/issue`
- Claiming or assigning as if this were `/solve`
