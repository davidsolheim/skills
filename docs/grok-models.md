# Grok-only models

Canonical spawn policy for every skill in this pack. Do not fork.

**Only Grok models.** Never Claude, GPT, Gemini, Composer, Cursor Auto, or
any other family — including when this skill pack is loaded from Cursor or
another host.

## Allowed slugs

| Role | `model` | When |
|------|---------|------|
| Default | `grok-4.6` | `/solve` implementer, optional `/solve` bug-only reviewer, nested `/solve`, `/issue` upgrades, `/prb` panel, `/prb` fixer, identify nested solve |
| Explore fan-out | `grok-4.5` | Read-only inventory / code-walk workers (`/project-review` deep explore, identify research-only upgrade workers). If the host rejects `grok-4.5`, use `grok-4.6` |

Spawn `subagent_type` (and the matching `~/.grok/roles/` + `~/.grok/agents/` entries) pins **reasoning** because `spawn_subagent` has no `effort` field. Authority: [`intensity.md`](intensity.md).

| `subagent_type` | Reasoning |
|-----------------|-----------|
| `solve-implementer` | low |
| `solve-reviewer` | low |
| `prb-reviewer` | medium |
| `prb-fixer` | medium |

If the host rejects a custom type, use `general-purpose` + `model: grok-4.6` and say so once.

Do **not** omit `model` to inherit the parent. Inheritance picks the host’s
default (often not Grok).

`resume_from` keeps the source agent’s model. Do not pass a different family
on resume.

## Spawn contract

Every `spawn_subagent` must include `model: grok-4.6` (or `grok-4.5` only for
the explore-fan-out row). For `/solve` / `/prb` workers also pass the
`subagent_type` from the table above. If a slug is rejected, pick the other
**Grok** slug. Never fall back off-family.

## Anti-patterns

- Omitting `model` “because the parent is already Grok”
- Passing a fake `effort:` field on `spawn_subagent` (the tool has none; use the role)
- Mixing Claude/GPT “for diversity” on the `/prb` panel (the panel is four
  Grok overlays, plus runtime proof)
- Spawning Cursor Auto / Composer for `/solve` workers
- Passing `inherit-parent` / `auto` as a model slug
