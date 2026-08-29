# Intensity

Canonical auto-dial for `/solve` and `/prb`. Do not fork. Do not add a slash
command. Skills **read this file**.

**User interface:** `/issue` thinks, `/solve` applies, `/prb` audits. No
intensity flags in normal use. `--effort N`, `--exhaustive-review`,
`--no-exhaustive`, and `--max-fix-cycles N` remain hidden overrides.

**Dial agents, not proof.** Runtime proof stays on for in-scope changes
([`prove-it-works.md`](prove-it-works.md)). Model stays `grok-4.6`. Sequential
`/solve` stays the default.

**Fail closed:** missing stamp or ambiguous class → bump **one band up**. Never
bump down. Do not spawn a classifier agent.

## Split (do not reverse)

| Skill | Job | Thinking |
|-------|-----|----------|
| `/issue` | Research + execution-ready contract | Paid once. Do not cheapen. |
| `/solve` | Apply that contract onto local `dev` | **Low.** One implementer. No review swarm. |
| `/prb` | Audit `origin/main...dev` before ship | **Medium** panel. Scratch findings. One fixer. One Linear comment. |

Construction review happens at `/prb`, not inside `/solve`. `/yeet` is the
explicit skip-the-panel path and still requires proof.

`spawn_subagent` has **no** `effort` argument. Pin child reasoning with the
user roles/agents in `~/.grok/roles/` and `~/.grok/agents/`:

| Spawn `subagent_type` | Reasoning | Used by |
|-----------------------|-----------|---------|
| `solve-implementer` | low | `/solve` implementer (sequential + fast workers) |
| `solve-reviewer` | low | `/solve` inner bug-only reviewer (heavy/critical only) |
| `prb-reviewer` | medium | `/prb` panel specialists |
| `prb-fixer` | medium | `/prb` closed-loop fixer |

If the host rejects a custom type, spawn `general-purpose` with `model: grok-4.6`
and say so once. `[models] default_reasoning_effort = "low"` is the inheritance
safety net. Do **not** pass a fake `effort:` field on `spawn_subagent`.

## Bands

| Band | Typical work | `/solve` | `/prb` panel | Exhaustive | Max fix cycles |
|------|--------------|----------|--------------|------------|----------------|
| **light** | Docs, comments, skills, copy-only, rename with no behavior | 1 implementer, **no** inner review | thoroughness only | off | **2** |
| **standard** | Isolated UI, one-file bug with a cheap test, local component | 1 implementer, **no** inner review | 4-agent once | off | **2** |
| **heavy** | Interactive UI, new logic, non-auth public API, shared helper with a few callers | 1 implementer + **1 bug-only** reviewer | 4-agent once | off | **2** |
| **critical** | Auth, gating, billing, tenancy, schema/migration, payments, secrets, blast-radius helpers | 1 implementer + **1 bug-only** reviewer (security-shaped prompt if auth/secrets) | 4-agent | **on** | **2** |

4-agent panel = thoroughness, security, rules, challenge. Thoroughness failure
is still fatal. Specialists may warn-and-continue.

A tiny CSS/copy tweak on a user-visible surface is **standard**, not light.
UI is in-scope for proof; it does not need an inner review swarm.

Linear **priority is a tie-break**, not the classifier. Urgent + money/auth
path → **critical** even if the title is small. A High copy fix is still light.

Ticket length is not intensity. Classify on **risk class**, not prose volume,
and not on whether a Runtime proof section exists (every good in-scope ticket
has one).

## Stamp (`/issue` writes; `/solve` and `/prb` read)

Required on every create/upgrade leaf. Place as `## Intensity` immediately
after `## Implementer contract`. Parse the first `Band:` line
(case-insensitive). Valid values: `light` `standard` `heavy` `critical`.

```markdown
## Intensity

- Band: standard
- Why: isolated UI; one-file component
- Proof: on
```

`Proof:` is `on` when the leaf is in-scope for [`prove-it-works.md`](prove-it-works.md),
else `n/a`. Proof is independent of band.

Do not omit the section. Do not invent a fifth band.

## Classify (no extra agent)

Walk **top to bottom**; first match wins. Ambiguous → next higher band.

1. **critical** if any of: auth, gating, billing, permissions, tenancy,
   schema/migration, payments, secrets, cookies/webhooks as auth, Stripe (or
   equivalent money), shared helper used broadly (blast radius). Path keywords
   in the code map: `auth`, `billing`, `stripe`, `payments`, `migrations`,
   `schema`, `middleware` (auth/session). Linear Urgent **and** a money/auth
   path.
2. **heavy** if interactive UI (empty/error/loading, new flow), new business
   logic, public API that is not auth/billing, or a shared helper with a few
   callers.
3. **standard** if isolated UI / one-file bug with a cheap local test / local
   component. Includes “tiny CSS” on a real route.
4. **light** if docs, comments, skill markdown, copy-only with no layout
   behavior, or a rename with no behavior change.

`/issue` (and `/issues`, `/project-review`, `/walk`, `/tidy` upgrades, `/identify`
upgrades) stamps after research. `/solve` does **not** re-litigate a valid stamp.

## `/solve`

`/solve --effort N` is a **hidden override for inner-review count**, not Grok
reasoning and not “run `/implement` with N reviewers.”

1. CLI `--effort N` (0–5) wins when present: `0`/`1` with light/standard → no
   inner review; `N >= 1` on heavy/critical → **one** bug-only reviewer.
   **Cap inner reviewers at 1.** Never spawn 2–6 construction reviewers.
2. Else read `## Intensity` → `Band:` → the Bands table above.
3. Else infer from the issue body + code map using Classify. Fail closed
   (bump up). Announce: `Intensity: <band> · inner-review none|bugs-only · proof on|n/a`.
4. Never default to an inner review swarm. Never run bundled `/implement`
   until-zero-nits. Standalone `/implement` is unchanged.
5. Thin tickets (no code map / AC / drift anchors): **upgrade via `/issue` on
   this id** (write the contract back) or skip. Do not implement a stub at low.

Fast workers inherit the **per-issue** band (inner-review yes/no). Do not apply
one inner-review policy to a mixed batch unless `--effort` was passed.

## `/prb` ship

```text
ship_intensity = max(stamps in origin/main...dev)
if the three-dot diff touches auth/billing/schema/migrations
   and ship_intensity is not already critical:
  bump to critical
```

Band order: light < standard < heavy < critical. One critical commit makes
the whole ship critical.

Then apply the Bands table for panel, exhaustive, and max fix cycles.
`--exhaustive-review` forces exhaustive on. `--no-exhaustive` forces it off.
`--max-fix-cycles N` wins over the table.

**Closed-loop:** do **not** file Linear issues or nested `/solve`. Fix from the
merged gate markdown via `prb-fixer`. One Linear comment when the gate finishes
([`../prb/references/linear-ship-comments.md`](../prb/references/linear-ship-comments.md)
template C). Leftovers at cap stay in that comment + the Phase 5 report.

**Babysit re-review:** re-run only specialists whose area changed. A copy fix
does not relaunch security + rules + challenge. Nits never block ship.

`--skip-review` still skips the panel (not proof). `/yeet` stays the explicit
skip-the-panel path.
