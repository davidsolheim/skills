# Intended State — Inferring “Done and Good” Without a Human List

Phase 1 builds an internal model of what the product is **supposed** to be so discovery can find gaps. The agent builds this model; the user does not have to dictate features.

---

## Goal

Produce a compact intended-state snapshot:

```text
primary_journeys: [...]
secondary_features: [...]
good_reference_surfaces: [...]   # already-good UI to match
constraints: [...]               # mobile-first, a11y, brand
confidence: high | medium | low
assumptions: [...]
out_of_scope_guesses: [...]      # explicitly not inventing these
```

This is **internal** (optional to show the user). It drives what you look for, not the final ticket text alone.

---

## Source priority (high → low)

1. **In-repo product specs** — `docs/prd.md`, SOW, `docs/product/`, design specs, Figma links in README
2. **AGENTS.md / CLAUDE.md / README** — product description, non-goals, stack constraints
3. **Linear history** — open epics/features + recent Done (what the team already believed mattered)
4. **Code as product truth** — nav labels, route titles, marketing site copy, onboarding steps
5. **User bias this turn** — “focus on onboarding”, Loom notes, pasted feedback (bias, not sole source)
6. **Domain defaults** — only when 1–5 are thin (e.g. SaaS apps usually need auth, empty states, settings)

Never invent a large product roadmap that the repo never implied.

---

## How to read each source

### Specs / PRD / SOW

- Extract named features and must-have journeys
- Note explicit non-goals (do not ticket those as missing)
- Capture acceptance language if already written (can seed Expected behavior later)

### Linear

- Open feature tickets ≈ intended but unfinished
- Open bugs ≈ known current failures (relate/dedup later; do not blindly re-file)
- Done tickets ≈ behavior that should still hold unless supersession says otherwise

### Code / nav

- If it’s in primary nav, treat as intended surface
- Onboarding checklists and empty-state CTAs declare intended next actions
- Feature flags: default-on paths are intended for “normal” users

### Good reference surfaces

Find 1–3 places that already look intentional (billing cards, settings forms, marketing).  
Taste tickets should **match these**, not invent a new design system.

---

## Confidence levels

| Level | When | Behavior |
|-------|------|----------|
| **High** | Clear PRD/nav + active Linear project | Discover aggressively against stated intent |
| **Medium** | Partial docs; strong code structure | Discover from code + nav; mark Assumptions on tickets |
| **Low** | Greenfield / almost no docs | Completeness tickets only for obvious half-built paths; prioritize functional/UI bugs over inventing features |

At **low** confidence, prefer:

- Broken / stubbed / placeholder paths
- Consistency with the few good screens that exist  
Avoid inventing “we should have a full CRM” without evidence.

---

## When to ask the user (once)

Ask **one** short question only if:

- Two contradictory intents appear (e.g. docs say “no billing”; nav has Billing half-built and you cannot tell which wins), **and**
- Filing either way would create a wrong multi-ticket queue

Otherwise state Assumptions on issues and proceed. Prefer agentic progress over interview mode.

---

## Mapping intended state → discovery

| Intended item | Discovery question |
|---------------|--------------------|
| Journey J | Does J work end-to-end? Empty/error branches? |
| Feature F in nav | Is F implemented or stubbed? |
| Constraint C (mobile-first) | Do primary journeys work on small screens? |
| Good surface G | Do other surfaces match G’s patterns? |
| Non-goal N | Do **not** file “add N” |

---

## Anti-patterns

- Waiting for the user to enumerate features before starting
- Treating every open Linear ticket as a finding to re-create
- Inventing enterprise features for a thin MVP without evidence
- Ignoring explicit non-goals in README/PRD
- Using marketing hype as literal AC without code pin
