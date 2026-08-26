# Prove it works

Canonical runtime-proof contract for `/solve`, `/prb`, and `/yeet`.
Do not fork. Do not add a slash command. Skills **read this file** and run it
inside their existing phases.

**Rule:** matrix green (typecheck, unit tests, build) is **necessary, not
sufficient** for in-scope changes. Do not mark In Review, push `origin/dev` for
ship, or merge to `main` on proxies, self-reports, or “it compiles.”

## In-scope (must drive the real path)

The change is in-scope if **any** of:

- User-visible UI, copy, layout, empty/error/loading states
- Auth, gating, billing, permissions, tenancy
- Public HTTP/API contract
- Schema / migration / shared helper used by more than the edited file

Out of scope: comments-only, docs-only, skill/markdown-only, internal rename
with no behavior change. Matrix from `AGENTS.md` is enough there.

## Proof standard

Check the **real artifact**, not a derived story:

1. Repo matrix from `AGENTS.md` / the ticket **Verification** commands.
2. Drive the feature the way a user (or caller) does.
3. Confirm the output: UI state, response body, DB row, file written — not
   “the handler looks right.”
4. Capture evidence (command output, screenshot path, response snippet). Keep
   it for the Linear closeout / `/prb` report. Do not commit secrets.

**Project-local verify skill (load if present, never invent a slash):**

Search the **current repo** (not `~/.grok/skills`) in order:

1. `.grok/skills/verify-*/SKILL.md`
2. `.cursor/skills/verify-*/SKILL.md`
3. `skills/verify-*/SKILL.md`

If found: follow its Launch / Doctor / Drive / Evidence / Cleanup. Prefer a
matching `features/*.md` map file for this ticket’s surface. If none exists:
drive from the ticket **Test plan** / **Runtime proof** + browser, curl, or CLI.
Do **not** scaffold a new verify skill mid-`/solve` or `/prb`.

## Visual parity (UI only)

When the ticket names a **reference surface**, screenshot, or “match X”:

- Drive the changed route **and** the named sibling (or inspect the screenshot).
- Compare hierarchy, spacing, empty/error, and the AC tokens.
- Loop until they match. One glance is not a pass.

If no reference is named: still drive the changed route; do not invent a new
look.

## Blast radius (shared helper, schema, auth/gating)

Grep of callers is not proof. Find the **one fact** the change is safe because
of. Prove it by **running code** (test or short script that imports the same
module the app ships). If you cannot get to a failing-loud run, write
**unproven** in the closeout and **do not** treat the gate as passed for that
class of change.

## Ticket kind (implementer — no extra slash)

| The issue is… | Do this while implementing |
|---------------|----------------------------|
| Bug with a cheap local test path | Failing test first, then the fix |
| User-visible UI | Match the named sibling; drive the route |
| Crosses a module boundary | Settle types / call shape before filling logic |
| Shared helper / schema / auth | Prove the one safety fact by running code |
| Anything in-scope | Runtime proof before claiming complete |

Parse and validate at **system boundaries** (HTTP, env, webhooks, external
JSON). Trust internal types after that. Do not nil-guard a crash and call it
fixed.

## Who runs this

| Skill | When | Fail action |
|-------|------|-------------|
| `/solve` | Phase 6 (and fast workers before they push the issue branch) | Do not merge to `dev` / do not In Review |
| `/prb` | Phase 1.5 **after** a clean review panel, **before** push. Independent of the panel. `--skip-review` skips the panel only, **not** this proof | Do not push / do not open PR |
| `/yeet` | After `origin/main` is in `dev`, **before** merge to `main` | Do not merge |

`--skip-review` does **not** waive runtime proof. Docs-only ships are out of
scope above.

## Closeout evidence (required when in-scope)

Linear completion / `/prb` report must include:

- Commands run + pass/fail
- What was driven (route, API, CLI) and the observed end state
- Visual reference compared, or `n/a`
- Blast-radius fact + proof, or `n/a` / `unproven` (unproven blocks in-scope)

## Rationalizations

| Excuse | Reality |
|--------|---------|
| "Tests passed" | Tests are a proxy. Drive the path. |
| "No browser tools" | Use curl, CLI, or the verify skill. If still impossible, **block** — do not pass. |
| "I'll note 'not performed'" | That is a fail, not a pass. |
| "Screenshot from before the fix" | Re-drive after the change. |
| "Call sites look fine" | Run the safety fact. |
| "Skip-review / yeet means skip proof" | Panel is optional on those paths. Proof is not, when in-scope. |
| "This is a tiny CSS tweak" | UI is in-scope. Drive the route. |

## Anti-patterns

- Declaring In Review / ship because typecheck passed
- Treating implementer summary as observation
- Generating a new verify skill or slash instead of driving this ticket
- Asking the user to click around as the only proof when tools can drive it
