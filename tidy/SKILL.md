---
name: tidy
description: >
  Hygiene pass on this repo's `.wcp/issues/` queue: upgrade thin issues to the
  /issue bar, retitle, cancel high-confidence duplicates, block real
  dependencies, and set `done` when a work commit already exists. Skip any
  issue tidied in the last 7 days unless /tidy --force or /tidy 0123.
  Finish the board first, then one needs-you list. Skip live foreign claims.
  Track last pass via a tidy-pass line in the issue file plus a local ledger.
  Use when the user runs /tidy, /tidy --force, /tidy TEAM-123, says "tidy
  the queue", "tidy the board", "clean up the board", "thicken thin tickets",
  or "close issues that are already done".
argument-hint: "[--force] [TEAM-123]"
---

# /tidy — Queue hygiene for `.wcp/issues/`

Inspect **`.wcp/issues/`** ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Do not call Linear. For every **due** issue: thicken thin
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
  second template. When thickening, fill **Occupancy (wcp)** from that
  template ([`../docs/wcp.md`](../docs/wcp.md)). Sharing a file is occupancy,
  not a reason to block the ticket.
- **Status** matches the folder. `done` means the writing is finished. `deployed-dev` means the commit is on `origin/dev`. `deployed-main` means it is on `origin/main`. `canceled` needs `reason`. `blocked` is only a real dependency.
- **High-confidence writes apply immediately**, including Cancel/Duplicate.
  Low-confidence closes are listed, not applied. Rules:
  [`references/actions.md`](references/actions.md).
- **Live foreign claim** → skip entirely. No rewrite, no status, no stamp,
  no cooldown. See `/solve` `references/multiplayer.md`.
- **Needs-you last.** Do not stop the pass. Do not set Blocked. Collect
  questions; ask once at the end.
- **No implement, no assign-for-solve, no `claimed-by:`.**
- **Secrets:** env **names** only.
- Edit the issue files first. Mirror the change when `.wcp/tracker.md` names a tracker. Skip when it is missing or `none`. Inventory is `open/`, `in-progress/`, and `blocked/`. Skip `done/`, `deployed-dev/`, `deployed-main/`, and `canceled/` unless the user names that id. Do not tidy an `in-progress/` issue whose `assignee` is someone else.

## Trigger phrases

`/tidy`, `/tidy --force`, `/tidy TEAM-123`, `tidy the queue`, `tidy the board`,
`clean up the board`, `thicken thin tickets`, `close issues that are
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
| (none) | Every **due** issue in this repo's `.wcp/issues/` queue (last pass ≥ 7 days ago or never) |
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

Set `ISSUE_SKILL_MD`, `SOLVE_SKILL_MD`, `SOLVE_SKILL_DIR`. Inventory fetch
lives in `$SOLVE_SKILL_DIR/references/eligibility.md`.

If **issue** is missing, still do status/dup/rollup; skip body upgrades and
mark those as not upgraded. If **solve** is missing, still skip anything with
a live wcp lease or ticket lease held by someone else; fall back to
`$HOME/.grok/skills/solve/references/eligibility.md` for the inventory fetch.

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

## Phase 1 — Queue

The board is `.wcp/issues/` in this checkout. Do not resolve a team or project.

---

## Phase 2 — Inventory

If `PINNED_ID` is set: open that issue file only. If it is `done` or `canceled`, report that and **stop**.

Otherwise list `open/`, `in-progress/`, and `blocked/`. Skip a live lease held by someone else. Load the local ledger ([`references/ledger.md`](references/ledger.md)) for the 7-day cooldown. Do not call Linear.

### Due vs skip

| Condition | Action |
|-----------|--------|
| `PINNED_ID` set and this is not that issue | Ignore (out of scope; never listed) |
| `in-progress/` whose `assignee` is someone else | **Skip claimed** — no writes |
| Last `tidy-pass` < 7 days and not `FORCE` and not pinned | **Skip cooldown** |
| Else | **Due** |

If nothing is due: report counts (cooldown / claimed) and **stop**. Do not
count Done/Canceled by listing the whole project.

---

## Phase 3 — Classify and act each due issue

For each due issue (lowest identifier number first; parents after their
children when both are due):

Follow [`references/actions.md`](references/actions.md) in this order:

1. **Claimed?** already filtered.
2. **Completed in git?** A work commit that meets acceptance sets the file `done`. Notion follows where that commit sits: `origin/main` → `done`; only `origin/dev` → `in-review`.
3. **Epic rollup?** all children terminal → file `done` + rollup note on the file.
4. **High-confidence duplicate / fully obsolete?** → `canceled` with `reason`
   (canonical or superseding id).
5. **Title** vague → retitle.
6. **Dependency** obvious → move the file to `blocked/` with `reason: blocked by <id>`.
7. **Thin body?** investigate like `/issue` Phase 3; update the **existing**
   issue to the `/issue` bar. Ready → leave body alone.
8. **Needs a human?** do not guess. Append to `NEEDS_YOU`. Leave state.
   Still stamp the pass so we do not re-ask for a week.

Apply high-confidence file and Notion updates as you go. Do not implement code.

If upgrade cannot reach the `/issue` bar without a product decision or a
code pin, do **not** save a half body. Record `needs-you` instead.

Stamp every due issue you actually processed (including inspect-only and
needs-you). Do **not** stamp claimed-skips or cooldown-skips.

---

## Phase 4 — Ledger

After the due set is processed, write:

1. The local ledger file (create parent dirs). Merge with previous entries. Do not post a tracker comment.

---

## Phase 5 — Report, then needs-you

```markdown
**Tidy:** `.wcp/issues/` · run <RUN_ID>
**Scope:** all due | force | pinned TEAM-123
**Due / cooldown-skip / claimed-skip:** D / C / K

### Changed
- TEAM-123 — upgraded · retitled · relatedTo TEAM-80
- TEAM-124 — `in-review` (`origin/dev` <sha>)
- TEAM-125 — `canceled` (`reason`: duplicate of TEAM-90)
- TEAM-100 — epic rollup `done`

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
issue files/assumptions (and status only if they explicitly confirm a
close). Do **not** re-scan the whole board unless they run `/tidy` again.

---

## Status writes

| Write | When |
|-------|------|
| Body / title | Thin or vague, after research |
| `blocked/` + `reason` | Hard dependency, naming the blocker id |
| `canceled` + `reason` | High-confidence duplicate or obsolete |
| Notion `in-review` | That file's commit is on `origin/dev` and not on `origin/main` |
| Notion `done` | That file's commit is on `origin/main` |
| Assignee / `in-progress` claim | **Never** |
| App source / git branch / push | **Never** |

---

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/tidy` | Board hygiene + stamps. No implement |
| `/stat` | Read-only briefing of the open board; no writes |
| `/issue` | Quality bar |
| `/identify` | Picks a small batch to **solve**; upgrades only that batch; JIT-claims on approve |
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
- Scanning `done/` / `canceled/` as if they were due
- Posting a tracker comment (the stamp is the `tidy-pass:` line in the issue file)
- Tidying or stamping Done / Canceled / Duplicate issues as if they were due
