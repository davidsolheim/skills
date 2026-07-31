# Decomposition & connectivity (`/issues`)

Turn a multi-item dump into atomic Linear leaves and decide how (or whether)
they connect.

---

## 1. Split the dump

| Signal | Treat as separate raw item |
|--------|----------------------------|
| Bullet / numbered list entry | Yes |
| Heading + paragraph | Yes (heading = theme, body may split further) |
| “Also / and / plus” joining two outcomes | Usually **two** items |
| Same outcome restated twice | **One** item |
| Nested sub-bullets under one feature | Often one leaf if single AC; split if each sub is shippable alone |

Prefer **over-split then merge** over mega-tickets. Merge only when two bullets
share the same acceptance criteria and same primary files.

---

## 2. Atomic leaf test

A leaf is ready when:

1. An implementer can mark it **Done** without waiting on undefined sibling work  
2. Acceptance criteria are checklist-testable on their own  
3. Code map points at a coherent primary package/app  
4. Title names **one** outcome  

Fail the test → split or rewrite.

**Bad:** “Fix billing UI and migrate research jobs and clean docs”  
**Good:** three leaves — billing UI, research job cutover, docs language.

---

## 3. Class taxonomy

| Class | Meaning | File early? |
|-------|---------|-------------|
| `foundation` | Schema, API shell, shared primitive others need | First |
| `feature` | User-visible capability | After foundations it needs |
| `a11y` | Accessibility (critical → treat like feature) | With features if blocking |
| `polish` | Hierarchy, motion, density, consistency | Later |
| `content` | Copy-only | Later |
| `chore` | Cleanup, renames, non-user debt | As priority warrants |

---

## 4. Connectivity

### Independent (default)

- Different domains/packages or no shared hard dependency  
- **Linear:** no parent, no `blockedBy`  
- Soft “same initiative” is **not** enough for an epic if items do not share delivery

### Soft related

- Same surface; either can ship alone  
- **Linear:** `relatedTo` after create  
- Example: two independent bugs on the same settings page

### Hard blocked (`blockedBy`)

Use only when **B’s AC cannot be met until A is done**:

- A adds a `tag_ids` column + API  
- B builds multi-select UI that **requires** that field  

Prefer soft ordering (file A first) when B could still ship a degraded path.

**Anti-pattern:** blocking polish on unrelated features so `/solve` starves.

### Epic cluster

Create a parent when **all** of:

1. ≥2 leaves will be filed  
2. They share one named initiative from the user dump  
3. Not a random residual grab-bag  

Examples that **deserve** an epic:

- “Billing cutover residual” with 4 related billing leaves  
- “Tags cutover” foundation + UI + backfill  

Examples that **do not**:

- “Random leftover bugs” across `apps/web` and `packages/api`  
- Two docs chores + one billing bug  

`--no-epic` forces flat. `--epic "Title"` forces one parent for all creates.

---

## 5. Monorepo ownership

Never put two packages’ runtime work in one leaf unless the ticket is an
explicit integration with AC on both sides.

| Package (example) | Typical leaf prefix |
|-------------------|---------------------|
| `apps/web/` | web / dashboard / marketing routes |
| `packages/api/` | API / handlers |
| `packages/db/` | schema / migrations |

Use the repo’s real paths and `AGENTS.md` language for titles and ownership.
Do not invent product brand names that are not in the dump or docs.

---

## 6. Duplicate & supersede

| Board match | Action |
|-------------|--------|
| Same AC / same surface open | **Skip create**; plan row = duplicate of `TEAM-n` (or real `PREFIX-n`) |
| Overlapping but extra scope | Create; `relatedTo`; note delta in Summary |
| This dump abandons old approach | Create; **Supersedes** full/partial on older ids |

---

## 7. Filing order

```text
1. foundation  (priority Urgent → Low)
2. feature / critical a11y
3. polish / content / chore
```

Then create blocked leaves after their blockers when possible so identifiers
and human scan order match the graph.

---

## 8. Temp id graph sketch

Before file:

```text
EPIC Billing cutover residual (optional)
  L1 foundation  P2  Add invoice line-item schema
  L2 feature     P2  Settings billing summary UI     related L1
  L3 feature     P1  Webhook retry for payments      blockedBy: (none if parallel)
  L4 chore       P3  Docs cleanup                    independent
```

After create, replace `L#` with real Linear identifiers (`PREFIX-N`, e.g. `TEAM-…`)
in relations and epic rollup.
