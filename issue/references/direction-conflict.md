# Direction-conflict check

Search **non-implemented** `.WCP/issues/` files for a **contradicting direction**
before filing. Do not leave both X and Y implementable. Notion follows
[`../../docs/notion-issues.md`](../../docs/notion-issues.md). Do not call Linear.

Duplicates are the same intent. Contradictions are **mutually exclusive**
intents for the same surface. Stack migrations are one subtype, not the whole
class.

Parent skills: [`../SKILL.md`](../SKILL.md) (`/issue`),
[`../../issues/SKILL.md`](../../issues/SKILL.md),
[`../../project-review/SKILL.md`](../../project-review/SKILL.md).

| Caller | Authority (who wins) |
|--------|----------------------|
| `/issue`, `/issues` | **This user message.** Older unstarted tickets lose. |
| `/project-review` | **Repo + board canonical direction.** An agent-invented finding does **not** beat an explicit user ticket of the opposite intent. |

`/project-review` classifies **offline** against the one board snapshot. Retire
only at publish (orchestrator). Workers never call Notion.

---

## Why this exists

`/identify` and `/solve 1` will pick an older Backlog/Todo ticket as written.
`## Supersedes` on the **new** ticket does not stop that. `/solve` S0 skip is
not the retire step.

Example: backlog “do X”, user now says “do Y instead” → file Y, **retire** X.

---

## When (mandatory)

- `/issue` Phase 2 and `/issues` Phase 2 **before** the deep writeup.
- `/project-review` board-sync (fast Phase 3 / deep D5) against the snapshot;
  retire in the publish pass (fast Phase 7 / deep D7).
- Intra-batch (`/issues` dump or review `final/` set): if both X and Y are in
  the same create set, **drop X** (`drop-contradicted`); file Y only.

`--draft` / `--plan-only`: classify and show retire/drop; do **not** change
file or Notion status until a real create succeeds.

---

## Search (actionable issues only)

Read `open/`, `in-progress/`, and `blocked/`. Skip `done/` and `canceled/`.

1. Match title and body against the new work’s surface: route, feature,
   component, product noun, stack, and the approach being abandoned.
2. Read the body only for hits that share that surface.
3. A live `in-progress` lease (`lease_expires` in the future, assignee not you)
   is not auto-retired.

`/issues`: one pass for the whole dump.

`/project-review`: use the local files already read for the board snapshot.
Do not re-read the whole queue per candidate.

---

## Classify

A **contradiction** exists when implementing both as written would produce two
conflicting product or architecture outcomes on the same surface.

| Class | Signal | Action |
|-------|--------|--------|
| Duplicate | Same surface + same outcome | Do **not** create; reply with existing id |
| Related, compatible | Same area, both can ship | Create; `relatedTo` |
| Full contradiction | Mutually exclusive (X vs Y; keep vs remove; stack A vs B) | File **this** ticket; **retire** unstarted X after create |
| Partial supersede | Y replaces part of X; residual AC remains | File Y; comment on X; leave X open only for residual |
| Partial, residual empty | After Y, X has nothing left to implement | Treat as full contradiction |
| Unrelated | Shared words only | Ignore |
| Ambiguous | Two plausible **new** directions; this message does not pick | Ask **once**; do not file; do not retire |

**Authority (`/issue`, `/issues`):** this user message wins over older open
tickets. Do not ask “cancel X?” when the user just stated Y.

**Authority (`/project-review`):** apply in order — (1) repo docs + code on
`main`/`dev`, (2) newer explicit direction tickets on the snapshot, (3) the
review finding only if (1) or (2) support it. If the finding contradicts an
explicit user ticket and repo truth does **not** pick the finding: **drop the
finding** (`drop-contradicted`); do **not** retire the user ticket. If
ambiguous: needs-you; do not file; do not retire.

**Unstarted** (auto-retire when full contradiction, high confidence): Backlog,
Todo, Triage, Ready, Unstarted, and team equivalents.

**Do not auto-retire:** In Progress with a live foreign `claimed-by:` or
assignee who is not you; In Review (already coded). List those as
**Conflict — needs you**. Still file Y.

Low confidence (“maybe related”) → `relatedTo` only; do not cancel.

---

## Retire (after the new issue exists)

Do this only after the new file exists. If that write fails, do not touch X.

For each unstarted full-contradiction file that is not under a live lease:

1. Set `status: canceled`, write `reason` naming the new id, clear the lease,
   and move the file to `canceled/` (player skill, Cancel).
2. Set that Notion row to `canceled`.
3. New ticket body: `## Supersedes` with the old id.

Do **not** assign, claim, or set `in-progress`. Do **not** set `done`.

---

## Create gate + reply

Fail closed if the search did not run.

User reply must include retire/conflict lines when any exist:

```markdown
**Retired:** [TEAM-40](url) — contradicted this direction (Canceled)
**Conflict — needs you:** [TEAM-55](url) — In Progress / foreign claim; not canceled
```

`/issues` plan `Action` values add: `drop-contradicted` (intra-batch),
`create + retire TEAM-n`.

`/project-review` handoff must list **Retired** and **Conflict — needs you**
the same way. Candidate index: `contradicts_board`, `retire_after_file`.

---

## Rationalizations

| Excuse | Reality |
|--------|---------|
| "`## Supersedes` + relatedTo is enough" | `/solve 1` on X will still implement X. Retire unstarted X. |
| "`/solve` / `/tidy` will cancel later" | Intake is the retire step. Later skills may never see Y in the same run. |
| "Cancel is not this skill" | This skill retires unstarted work that contradicts the ticket it files. |
| "Too slow / rapid-fire" | Search + retire is part of filing, not optional polish. |
| "Don't cancel other people's tickets" | Unstarted backlog is the user's board. Live foreign claims: escalate, don't cancel. |
| "Ask whether Y replaces X" | `/issue`/`/issues`: they just said Y. `/project-review`: ask only when repo+board do not pick. |
| "Review must never cancel tickets" | Mass-cancel is forbidden. Targeted retire of unstarted full contradictions is required. |
| "The review finding always wins" | Only if repo/board canonical agrees. Else drop the finding; keep the user ticket. |
| "X might have residual value" | If residual AC remains, partial + comment. If not, retire. |
| "Chat mention is enough" | `/identify` does not read chat. The file and the Notion row must change. |

## Red flags — STOP

- About to file Y while unstarted X still specifies the opposite approach
- About to rely on `/solve` S0 / batch guidance to skip X
- About to skip the board search because this is not a stack migration
- About to leave Backlog/Todo X open "for context"
- About to cancel In Progress (foreign claim) or In Review without asking
- About to cancel a user ticket because a review *invented* the opposite
- About to mass-cancel the open board from `/project-review`

**All of these mean:** search, then retire or escalate — then file.
