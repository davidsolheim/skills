# Dependency Ordering

Phase 6 (fast) / Phase D6 (deep) shapes the queue so `/solve` can drain it without implementing polish before foundations.

**Deep:** run this graph only on cleaned `issue-candidates/final/` candidates (and `_merged/index.json` `blocked_by_candidates`). Do not invent deps from raw `_inbox` dumps.

---

## Why this matters

`/solve` defaults to:

1. Pick the lowest-numbered eligible **open** leaf
2. Skip files in `blocked/` whose `reason` names an unfinished blocker
3. Multi-issue runs may apply batch guidance for platform conflicts
4. Lowest issue **number** is the usual tie-breaker among independent leaves

Review should therefore:

- Put **foundations first** when filing (lower numbers)
- Write hard dependents in `blocked/` with `reason: blocked by <id>`
- Put an initiative name in the leaf body when a batch needs one. There is no epic file.

---

## Leaf class taxonomy

| Class | Meaning | Typical lenses |
|-------|---------|----------------|
| `foundation` | Shared primitive, API, schema, or route shell others need | Completeness, Functional |
| `feature` | User-visible capability on top of foundations | Completeness, Functional, Edge |
| `polish` | Hierarchy, motion, density, consistency | UI, Hierarchy, Taste |
| `content` | Copy-only | Content |
| `a11y` | Accessibility fix (may block if legal/critical → treat as feature) | A11y |

Assign `class` in Review metadata on each draft.

---

## Hard vs soft dependencies

### Hard (`blocked/` + `reason: blocked by <id>`)

B’s acceptance criteria **cannot** be met until A is done, e.g.:

- A: Add `/projects` list API + page shell  
- B: Empty state for `/projects` list  
- A: Introduce shared `Modal` animation primitive  
- B: Apply that modal motion on Settings (if B’s AC requires the shared primitive)

### Soft (filing order only, no blocked reason)

- Nice to fix foundation first but B is independently shippable
- Taste polish that does not require a new component system

Prefer soft ordering when unsure — over-blocking stalls `/solve`.

---

## Filing order

Create in this order:

```text
1. foundation (P0 then P1 then P2)
2. feature    (P0 then P1 then P2)
3. a11y critical / content blocking primary journeys
4. polish + remaining content + remaining a11y
```

Secondary sort: higher priority first within a class.

This biases queue identifiers so default lowest-number selection starts in a sensible place when no hard `blocked by` graph exists.

---

## Initiative packaging

### Default

Put this initiative name in each leaf body for the run:

```text
Review pass ({mode}) – {Project or Surface} – YYYY-MM
```

Examples:

- `Review pass (fast) – Teton Web Platform – 2026-07`
- `Review pass (deep) – Onboarding – 2026-07`

See `issue-template.md`. There is no epic file.

### `--no-epic`

Skip the initiative line. Still use filing order + `reason: blocked by <id>`.

### Multiple surfaces in one deep run

Prefer **one initiative name** with clear surface prefixes in leaf titles, unless surfaces are huge and unrelated — then one initiative name per major surface.

---

## Graph sketch (internal)

Before filing:

```text
Review pass (deep) – App – 2026-07
  L1 foundation  P0  Fix projects API 404
  L2 feature     P1  Render projects list      blocked by L1
  L3 feature     P1  Empty state for projects  blocked by L2
  L4 polish      P2  Hierarchy on projects card blocked by L2
  L5 content     P2  Rewrite empty-state copy  blocked by L3
```

After create, replace L# with real ids in `reason: blocked by <id>`.

---

## Interaction with `/solve`

| Review action | Solve behavior |
|---------------|----------------|
| Initiative name on leaves | Implement lowest eligible open leaf |
| `reason: blocked by <id>` | Skip until blocker `done` or `canceled` |
| Foundation filed first | Likely lower number → earlier pick |
| Unassigned `open/` | Eligible for claim |
| In Progress / assigned others | Not set by review (good) |

---

## Anti-patterns

- One mega ticket with a huge AC list
- All polish tickets numbered before foundation (reverse filing order)
- `blocked by` webs so dense nothing is eligible
- Using `blocked/` for soft sequencing
- Implementing dependency order only in handoff prose without `reason: blocked by <id>`
