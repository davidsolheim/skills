# Dependency Ordering & Epic Packaging

Phase 6 (fast) / Phase D6 (deep) shapes the queue so `/solve` can drain it without implementing polish before foundations or epic shells.

**Deep:** run this graph only on cleaned `issue-candidates/final/` candidates (and `_merged/index.json` `blocked_by_candidates`). Do not invent deps from raw `_inbox` dumps.

---

## Why this matters

`/solve` defaults to:

1. Expand epics → lowest-numbered eligible **child**
2. Skip blocked (`blockedBy` unfinished)
3. Multi-issue runs may apply batch guidance for platform conflicts
4. Lowest issue **number** is the usual tie-breaker among independent leaves

Review should therefore:

- Put **foundations first** when filing (lower numbers)
- Set **`blockedBy`** for hard dependencies
- Keep parents as **epics only** (never the only implementable unit)

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

### Hard (use `blockedBy`)

B’s acceptance criteria **cannot** be met until A is done, e.g.:

- A: Add `/projects` list API + page shell  
- B: Empty state for `/projects` list  
- A: Introduce shared `Modal` animation primitive  
- B: Apply that modal motion on Settings (if B’s AC requires the shared primitive)

### Soft (filing order only, no blockedBy)

- Nice to fix foundation first but B is independently shippable
- Taste polish that does not require a new component system

Prefer soft ordering when unsure — over-blocking stalls `/solve`.

---

## Filing order

Within an epic (or flat list), create in this order:

```text
1. foundation (P0 then P1 then P2)
2. feature    (P0 then P1 then P2)
3. a11y critical / content blocking primary journeys
4. polish + remaining content + remaining a11y
```

Secondary sort: higher priority first within a class.

This biases Linear identifiers so default lowest-number selection starts in a sensible place when no hard `blockedBy` graph exists.

---

## Epic packaging

### Default

One parent epic per review run:

```text
Review pass ({mode}) – {Project or Surface} – YYYY-MM
```

Examples:

- `Review pass (fast) – Acme Product – 2026-07`
- `Review pass (deep) – Onboarding – 2026-07`

Epic body: short packaging note only (see `issue-template.md`).  
**Never** put implementable AC only on the epic.

### `--no-epic`

File flat leaves. Still use filing order + `blockedBy`.

### Multiple surfaces in one deep run

Prefer **one epic** with clear surface prefixes in leaf titles, unless surfaces are huge and unrelated — then one epic per major surface.

---

## Graph sketch (internal)

Before filing:

```text
EPIC Review pass (deep) – App – 2026-07
  L1 foundation  P0  Fix projects API 404
  L2 feature     P1  Render projects list      blockedBy: L1
  L3 feature     P1  Empty state for projects  blockedBy: L2
  L4 polish      P2  Hierarchy on projects card blockedBy: L2
  L5 content     P2  Rewrite empty-state copy  blockedBy: L3
```

After create, replace L# with real identifiers in `blockedBy`.

---

## Interaction with `/solve`

| Review action | Solve behavior |
|---------------|----------------|
| Epic with children | Expand; implement lowest eligible child |
| `blockedBy` open | Skip until blocker Done |
| Foundation filed first | Likely lower number → earlier pick |
| Unassigned Backlog/Todo | Eligible for claim |
| In Progress / assigned others | Not set by review (good) |

---

## Anti-patterns

- Single epic with no children and a huge AC list
- All polish tickets numbered before foundation (reverse filing order)
- `blockedBy` webs so dense nothing is eligible
- Using Blocked **state** instead of `blockedBy` for normal sequencing
- Implementing dependency order only in handoff prose without Linear relations
