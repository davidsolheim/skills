# Identify thin S0 (platform order)

Run this **after** eligibility and **before** ranking. Vocabulary and conflict
rules are `/solve` batch guidance — do not invent a second direction policy.

Authority: `$SOLVE_SKILL_DIR/references/batch-guidance.md` (tag, conflict,
classify, canonical vs abandoned).

Identify does **not** write `guidance.md` / `graph.json` unless direction
confidence is **low** (then stop and ask). Identify does **not** comment,
Cancel, or Duplicate during this pass.

---

## When there is no platform conflict

If open leaves + repo docs do not disagree on stack (no abandoned vs canonical
split):

- Drop only **full-obsolete** leaves you are high-confidence would be
  guidance-`skip` (record in “Not in this batch”).
- **Do not** promote every “foundation” over Urgent user-facing bugs.
- Proceed to ranking.md with remaining `ELIGIBLE`.

---

## When there is a conflict

Tag each `ELIGIBLE` leaf (class + platforms + explicit `## Supersedes`) using
batch-guidance.md Step B–E.

Then:

| Action | Identify does |
|--------|----------------|
| `skip` (full obsolete / abandoned stack, no remaining product value) | Remove from `ELIGIBLE`. No Linear write. |
| `promote` (open migration / canonical foundation) | Band 0 — sort before features that would be rewritten. |
| `rescope` (outcome still needed on canonical stack) | Keep in `ELIGIBLE`. Note “re-scope on solve”. Do not rewrite AC here unless the ticket is in `PROPOSED` and upgrade would anyway. |
| `normal` | Band 1. |

If two open tickets claim **conflicting new** architectures and repo truth is
weak: **stop**, show a conflict table, ask once. Do not rank yet.

---

## Sort bands (only when conflict)

1. Band 0 — `promote` migrations / canonical foundations
2. Band 1 — `normal` and `rescope` (rescope after its migration when both are
   eligible; otherwise P+U)
3. Inside a band: ranking.md (priority, then U, then number)

A High feature on an abandoned stack must not outrank an open canonical
migration.

---

## Linear

No comments, status, or Cancel from thin S0. `/solve` S0 / `/tidy` own those
writes. Mention skipped obsolete ids under “Not in this batch”.
