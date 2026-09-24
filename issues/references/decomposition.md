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
3. Code map points at a coherent primary package  
4. Title names **one** outcome  
5. Body is **self-contained** for a cheap model (plan + file map + contracts; not “see L1”)  
6. Scope fits one focused session (not a multi-day rewrite)

Fail the test → split or rewrite. Depth bar:
[`../../issue/references/execution-ready-bar.md`](../../issue/references/execution-ready-bar.md).

**Bad:** “Fix agents cost UI and migrate research to Perplexity and clean docs”  
**Good:** three leaves — cost UI, Perplexity cutover, docs language.

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

- A adds `product_category_ids` column + API  
- B builds multi-select UI that **requires** that field  

Prefer soft ordering (file A first) when B could still ship a degraded path.

**Anti-pattern:** blocking polish on unrelated features so `/solve` starves.

### Epic cluster

Create a parent when **all** of:

1. ≥2 leaves will be filed  
2. They share one named initiative from the user dump  
3. Not a random residual grab-bag  

Examples that **deserve** an epic:

- “Student success residual” with 4 related student CRM leaves  
- “Product categories cutover” foundation + UI + backfill  

Examples that **do not**:

- “Random leftover bugs” across app and services  
- Two docs chores + one billing bug  

`--no-epic` forces flat. `--epic "Title"` forces one parent for all creates.

---

## 5. Occupancy (WCP)

`/solve` runs many leaves on one local `dev`. Exclusive file leases:
[`../../docs/wcp.md`](../../docs/wcp.md).

- Give each leaf **one primary write path** (and symbol). Put it in
  **Occupancy (WCP)** on the body.
- Prefer splits whose primary paths **differ**, so two `/solve` workers can
  hold leases at once.
- Sharing a file is occupancy (next wave), not a reason to merge tickets.
- Mint Linear `blockedBy` only when B’s AC is impossible until A lands.
- Keep lockfiles / generated clients / root schema off most leaves. A leaf
  that must write a barrel says so under Occupancy.

## 6. Monorepo ownership

Never put two packages’ runtime work in one leaf unless the ticket is an
explicit integration with AC on both sides.

| Package | Typical leaf prefix |
|---------|---------------------|
| `app/` | app / dashboard / CRM routes |
| `services/` | Services |
| `agents/` | Agents / Eve / company-research |

Respect package `AGENTS.md` language (e.g. do not brand app as “CRM”).

---

## 7. Duplicate & supersede

| Board match | Action |
|-------------|--------|
| Same AC / same surface open | **Skip create**; plan row = duplicate of `TEAM-n` |
| Overlapping but extra scope | Create; `relatedTo`; note delta in Summary |
| This dump abandons old approach | Create; **retire** unstarted contradicted ids ([direction-conflict.md](../../issue/references/direction-conflict.md)); `## Supersedes` + board action |

---

## 8. Filing order

```text
1. foundation  (priority Urgent → Low)
2. feature / critical a11y
3. polish / content / chore
```

Then create blocked leaves after their blockers when possible so identifiers
and human scan order match the graph.

---

## 9. Temp id graph sketch

Before file:

```text
EPIC Student success residual (optional)
  L1 foundation  P2  Verify data model residual
  L2 feature     P2  Coach My View residual     related L1
  L3 feature     P1  Engagement automation      blockedBy: (none if parallel)
  L4 chore       P3  Docs cleanup               independent
```

After create, replace `L#` with the issue id in `reason` and in the leaf body. There is no epic file.
