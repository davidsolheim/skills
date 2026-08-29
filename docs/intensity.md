# Intensity

Canonical auto-dial for `/solve` and `/prb`. Do not fork. Do not add a slash
command. Skills **read this file** and set reviewer/panel width from it.

**User interface:** `/solve` then `/prb`. No intensity flags in normal use.
`--effort N`, `--exhaustive-review`, `--no-exhaustive`, and `--max-fix-cycles N`
remain hidden overrides.

**Dial agents, not proof.** Runtime proof stays on for in-scope changes
([`prove-it-works.md`](prove-it-works.md)). Model stays `grok-4.6`. Sequential
`/solve` stays the default. Intensity only changes how many reviewers argue
about the code, and whether `/prb` runs a second panel.

**Fail closed:** missing stamp or ambiguous class → bump **one band up**. Never
bump down. Do not spawn a classifier agent.

## Bands

| Band | Typical work | `/solve` effort | `/prb` panel | Exhaustive | Max fix cycles |
|------|--------------|-----------------|--------------|------------|----------------|
| **light** | Docs, comments, skills, copy-only, rename with no behavior | **1** | thoroughness only | off | **2** |
| **standard** | Isolated UI, one-file bug with a cheap test, local component | **2** | 4-agent once | off | **2** |
| **heavy** | Interactive UI, new logic, non-auth public API, shared helper with a few callers | **3** | 4-agent once | off | **2** |
| **critical** | Auth, gating, billing, tenancy, schema/migration, payments, secrets, blast-radius helpers | **5** | 4-agent | **on** | **2** |

4-agent panel = thoroughness, security, rules, challenge. Thoroughness failure
is still fatal. Specialists may warn-and-continue.

A tiny CSS/copy tweak on a user-visible surface is **standard**, not light.
UI is in-scope for proof; it does not need six reviewers.

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

1. CLI `--effort N` (1–5) wins when present. Apply to every issue in that run.
2. Else read `## Intensity` → `Band:` → effort from the table.
3. Else infer from the issue body + code map using Classify. Fail closed
   (bump up). Announce: `Intensity: <band> (effort N) · <why> · proof on|n/a`.
4. Never default to effort 5 unless the band is **critical** (or `--effort 5`).

Fast workers inherit the **per-issue** band. Do not apply one effort to a mixed
batch unless `--effort` was passed.

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

**Closed-loop `/solve`:** use the **finding’s** band, default **standard**
(effort 2), unless the finding itself is critical (effort 5). Do not inherit
ship-critical for a localized empty-state fix. Stamp the filed Linear issue.

**Babysit re-review:** re-run only specialists whose area changed. A copy fix
does not relaunch security + rules + challenge. Nits never block ship.

`--skip-review` still skips the panel (not proof). `/yeet` stays the explicit
skip-the-panel path.
