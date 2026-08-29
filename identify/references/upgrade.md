# Identify upgrade (thin → execution-ready)

Upgrade happens **only** for issues in the current `PROPOSED` batch, **before**
the user sees the approve prompt. Do not upgrade the rest of the board.

Authority for the quality bar is the **`/issue` skill**, not this file:

1. Read `$ISSUE_SKILL_MD` (draft / quality / create-gate sections).
2. Read `$ISSUE_SKILL_DIR/references/` templates if present
   (`issue-body-template.md`, `execution-ready-bar.md`).
3. Write the **existing** Linear issue. **Do not create** a second issue.

---

## Progress (required)

Before Linear writes, show a short status (todo or one message), **not** the
approve prompt:

```text
Identify: upgrading TEAM-… · TEAM-…  (skip ready / tidy-fresh)
```

Then upgrade. Then Phase 5 present.

---

## Thin vs ready

Treat as **ready** (no Linear write) when the body already has all of:

- Code map with paths that exist in **this** workspace
- Checklist acceptance criteria (pass/fail, not “improve UX”)
- Verification commands that exist in this repo (`AGENTS.md` / package scripts)
- Drift-check anchors (≥3)
- An ordered plan or file-by-file change list

Missing `## Intensity` alone does **not** make a ticket thin. When you **do** rewrite a thin ticket, stamp `## Intensity` per [`../../docs/intensity.md`](../../docs/intensity.md).

Treat as **thin** if any of those are missing, or the body is product prose
without paths/symbols.

---

## Tidy-stamp skip

If `/tidy`’s Linear stamp is fresh (**< 7 days**, parse
`$IDENTIFY_SKILL_DIR/../tidy/references/ledger.md` or
`$HOME/.grok/skills/tidy/references/ledger.md`) **and** the ready checks still
pass: **do not rewrite**. Mark **Ready:** `already ready (tidy)`.

If the stamp is fresh but the body still fails ready checks: upgrade anyway.

Do not require the local tidy ledger; Linear stamp wins.

---

## Parallel research, serial save

For each **thin** id in `PROPOSED`:

1. `spawn_subagent` (`general-purpose` or `explore`), `isolation: none`,
   **`model: grok-4.5`** if explore / research-only, else **`grok-4.6`**
   ([`../../docs/grok-models.md`](../../docs/grok-models.md)). Do not inherit.
   Read-only on app code. Prompt: `/issue` Phase 3 investigation + draft a full
   `/issue`-bar body for this existing id. Return title + markdown body +
   whether create-gate passes. No Linear writes from the worker.
2. Orchestrator **owns** Linear save. Fail-closed: if the gate fails, do **not**
   leave a half-rewritten body; drop from `PROPOSED` and pull the next ranked
   eligible leaf (then upgrade that one).

Ready / tidy-fresh tickets skip spawn.

Keep the original title unless it is vague (“Fix bug”); then retitle in Linear.
Keep Linear **priority** unless the ticket is Urgent/High in name only and the
body is clearly Low chore — do not silently demote user-facing P1/P2.
Preserve project, parent, labels unless a label is objectively wrong.

Record upgraded ids in `UPGRADED`.

---

## Drop from this batch

Drop and replace when:

- No code pin exists after a reasonable search
- The work is a product decision you cannot pre-decide in Assumptions
- The ticket is a parent/epic that slipped through (expand or skip)

Do not create follow-up issues during Identify. Mention the drop in
“Not in this batch”.

---

## Secrets

Env **names** only. No tokens, connection strings, Doppler values.

## What not to do

- Do not file a new issue “because the old one is messy”
- Do not upgrade tickets that are not in `PROPOSED`
- Do not implement the fix
- Do not change status or assignee here (claim is Phase 7, after approve, JIT)
