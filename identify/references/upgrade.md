# Identify upgrade (thin → execution-ready)

Upgrade happens **only** for issues in the current `PROPOSED` batch, **before**
the user sees that batch. Do not upgrade the rest of the board.

Authority for the quality bar is the **`/issue` skill**, not this file:

1. Read `$ISSUE_SKILL_MD` (draft / quality / create-gate sections).
2. Read `$ISSUE_SKILL_DIR/references/` templates if present
   (`issue-body-template.md`, `execution-ready-bar.md`).
3. Write the **existing** Linear issue. **Do not create** a second issue.

## Thin vs ready

Treat as **ready** (no Linear write) when the body already has all of:

- Code map with paths that exist in **this** workspace
- Checklist acceptance criteria (pass/fail, not “improve UX”)
- Verification commands that exist in this repo (`AGENTS.md` / package scripts)
- Drift-check anchors (≥3)
- An ordered plan or file-by-file change list

Treat as **thin** if any of those are missing, or the body is product prose
without paths/symbols.

## Upgrade procedure (thin)

1. Investigate like `/issue` Phase 3 (read-only on app code): grep, read
   primary files, one level of call chain, mirror pattern, real verify commands.
2. Draft a full `/issue`-bar body. Keep the original title unless it is vague
   (“Fix bug”); then retitle in Linear.
3. Keep Linear **priority** unless the ticket is Urgent/High in name only and
   the body is clearly Low chore — do not silently demote user-facing P1/P2.
4. Update via Linear save/update **with the existing id**. Preserve project,
   parent, labels unless a label is objectively wrong.
5. Create-gate (fail closed) — same checks as `/issue` before save. If the gate
   fails: **do not** leave a half-rewritten body; either finish the research or
   drop the ticket from `PROPOSED` and pull the next ranked leaf.
6. Record the id in `UPGRADED`.

## Drop from this batch (do not block Identify)

Drop and replace (Phase 4 of `SKILL.md`) when:

- No code pin exists after a reasonable search
- The work is a product decision you cannot pre-decide in Assumptions
- The ticket is a parent/epic that slipped through (expand or skip)

Do not create follow-up issues during Identify. Mention the drop in
“Not in this batch”.

## Secrets

Env **names** only. No tokens, connection strings, Doppler values.

## What not to do

- Do not file a new issue “because the old one is messy”
- Do not upgrade tickets that are not in `PROPOSED`
- Do not implement the fix
- Do not change status or assignee here (claim is Phase 7, after approve)
