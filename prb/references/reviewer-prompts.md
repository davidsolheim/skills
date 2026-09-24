# /prb — Reviewer specialist prompts (Phase 1.5)

Orchestrator-only. Prepend [`review-rubric.md`](review-rubric.md) (full text) to every prompt. Then append **one** specialist overlay below. Do not paste these overlays into `SKILL.md`.

Every spawn:

- `subagent_type`: `prb-reviewer` (fallback `general-purpose` if the host rejects the type)
- `model`: `grok-4.6` (required; never omit / never inherit; [`../../docs/grok-models.md`](../../docs/grok-models.md))
- `background`: `true` (launch the panel in one turn)
- `description`: `[thoroughness] …` / `[security] …` / `[rules] …` / `[challenge] …`
- Do **not** pass `capability_mode`. Reviewers must write their scratch file. Read-only is enforced by the prompt: no source edits, no git writes, no Linear, no push.
- Do **not** pass a fake `effort:` field. High reasoning comes from the `prb-reviewer` role.
- **Never** replace this file with a shorter stub. Exhaustive pass 2 and babysit use the same rubric + overlay + shared tail. Review target is always `origin/main...dev`.

Inline absolute paths for the rubric, ship-set commands, `AGENTS.md` excerpts, and `output_file`. Do not rely on shell variables surviving across tool calls.

Shared tail (append after every overlay):

```
Ship set (review only origin/main...dev on local dev):

  git log origin/main..dev --oneline
  git -c core.quotepath=false diff origin/main...dev
  git -c core.quotepath=false diff --name-only origin/main...dev

Repo: <owner/repo>
dev SHA: <sha>
origin/main SHA: <sha>
Changed paths:
<list>

Applicable AGENTS.md Code Review Rules (root + nested for changed paths):
<verbatim excerpts or "none found">

Known findings from earlier rounds (do not re-report unless still present and unfixed):
<list or none>

Write ONLY valid JSON matching the rubric schema to:
<output_file>

Do not edit project source. Do not commit. Do not push. Do not call Linear.
If you run tests, run a relevant subset only; do not use failures that already exist on origin/main as findings.
```

---

## Thoroughness

Focus: correctness, regressions, control flow, incomplete call-site updates, races, performance cliffs introduced by this diff, and whether claimed behavior matches the patch.

Walk the rubric’s always-actionable classes that are **not** primarily authz: N+1 / repeated identical loads in a loop this ship added, payload amplification, `||` restoring a cleared empty string, validate-source instead of validate-rendered, persist-then-ack races.

Stay on those issues. Do not run a second security audit or a second rules pass — other reviewers own those.

Work at high thoroughness: read surrounding source, follow types and callers, and run relevant tests when they exist.

---

## Security

Focus: exploitable issues introduced by this ship — authz bypass, secret leakage, injection, unsafe deserialization, SSRF, CSRF on state-changing routes that require it, unsafe defaults, PII in logs, **missing parse/validate at system boundaries** (HTTP, env, webhooks, external JSON) so illegal states leak into business logic.

**Construct the path.** Name who, which URL or body, and which branch grants extra access. A guessed `/api/files/…` URL, a planted status post, or an admin save is enough. Ownership vs readability is a finding: `authorize*Access` returning allow for “can I read this now” is not permission to attach, publish, or widen.

ACL ordering: a new reference must not un-gate an older, tighter rule (legacy message, connections-only post, unpublished notice). Author / live-post bypasses are findings unless the ticket pins them.

Always-actionable classes in the rubric are **P2 or higher**, not P3. Omit only defense-in-depth with **no** constructed path.

Do not review style, tests-as-tests, or unrelated architecture.

---

## Rules

Focus: `## Code Review Rules` (and equivalent review-invariant sections) in root and nested `AGENTS.md` that cover changed files.

For each finding: cite the rule file and smallest supporting line range. Include a safe path when the rule states one.

Do not invent rules. Do not flag generic bugs that are not rule violations — other reviewers own those. If no rules apply, return zero findings.

---

## Challenge

Focus: pressure-test the ship the way an external PR bot will.

- Does the diff actually implement what the commits / PR description claim?
- New behavior with no tests when this repo expects them; deleted tests without replacement
- Risky behavior changes (API contract, data, auth, migrations) that a careful reviewer would challenge
- Failure modes the author likely missed (empty input, partial failure, retries, rollback)
- **Missing proof:** in-scope UI/auth/billing/API/schema/shared-helper ships with no runtime evidence (tests-only / “looks correct”). Flag as P1 when the diff claims behavior the author did not drive.
- Walk every always-actionable class in the rubric against this diff. Those are **never nits**. If Challenge finds one, it stays P2+ through merge.

Do not duplicate a generic correctness nit or a generic security nit already in the rubric unless you can show a distinct defect. Prefer “claimed vs actual”, missing proof, and leftover always-actionable holes over style.
