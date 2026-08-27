# Finding kinds — what “every” means

`/walk` files **every actionable user-visible finding**. It does not copy
`/project-review` fast-mode’s P0/P1-only filter. Low-priority ideas still file
when they pass the create gate.

---

## Kinds

| Kind | When | Linear-ish type | Default priority |
|------|------|-----------------|------------------|
| **bug** | Something is wrong or broken relative to what this UI already promises | Bug | P0–P1 if a primary path; else P1–P2 |
| **improvement** | It works, but worse than a sibling pattern, design system, or obvious UX | Enhancement / polish | P1 high-visibility; else P2 / Low |
| **idea** | A capability this surface reasonably implies and does not offer | Feature | Usually Low; Medium if it unblocks a core loop |

Record `kind` on every candidate.

---

## File (actionable)

A finding is actionable when **all** of these hold:

1. A specific surface (route or named view)
2. Observable current behavior from the walk (or blocked load)
3. A concrete expected behavior (copy, layout, flow, or match-surface X)
4. A code pin (path + symbol, or the route file as anchor)
5. Independently shippable AC (not “clean up the marketing site”)

Examples that **must** file:

- CTA does nothing / goes to 404
- Console uncaught error on a user path
- Form accepts invalid input with no message
- Empty state is a blank hole
- Mobile nav has no way to reach a primary destination
- Button style on `/studio` does not match `/directory` primary button
- Public page implies “save” with no signed-in path (idea or improvement)
- “Export CSV” would complete the list page (idea, with a proposed control
  matching an existing export if one exists)

Split compounds: dead submit + unrelated footer 404 = two leaves.

---

## Drop (not a Linear leaf)

Drop and optionally mention in handoff **Coverage notes**, not as tickets:

| Drop | Why |
|------|-----|
| “Page loaded and looked fine” | Not a finding |
| Duplicate of another finding this run | Merge evidence into the first leaf |
| High-confidence duplicate of an **open** board issue | `board_match`; skip create |
| Vague taste that cannot pass `taste-to-concrete` | Not executable |
| Backend/job/schema gap with **no** UI surface | `/project-review`, not this skill |
| Third-party outage you cannot pin to app code | Note in handoff |
| Pure nit below an independent change (one letter kerning, 1px) | Only keep if it is a documented token violation you can name |
| Finding that contradicts an explicit **user** ticket | `drop-contradicted` |

Do **not** drop because:

- Priority would be Low
- It is “just an idea”
- The walk already filed many bugs
- `/project-review` would have pruned it in fast mode

---

## Ideas still need a plan

For `idea` leaves, the intake agent **pre-decides**:

- Where the control lives (exact surface)
- What it looks like (mirror component X)
- What happens on success/empty/error
- What is out of scope (no new product line)

If you cannot pre-decide without a human product call, put that under Risks and
**do not file** a pretend-ready ticket. Rare. Prefer a small assumed slice.

---

## Priority heuristics

| P | Examples |
|---|----------|
| **P0** | Core journey blocked, data loss, auth hole on a public control, blank primary page |
| **P1** | High-traffic broken secondary, ignored 500 on save, unusable mobile primary nav, missing empty state on a core list |
| **P2** | Consistency vs sibling, copy, a11y on secondary, most ideas |

Linear map: P0 core → `1`; other P0 / visible P1 → `2`; remaining P1 / solid P2
→ `3`; ideas and minor improvements → `4`.

---

## User report shape

Quote the walk, not a slogan:

> On `/` at 375px, tapping **Studio** in the bottom row does nothing. Console:
> `TypeError: ...` after click. Expected: navigate to `/studio`.

Improvements/ideas: same shape — current vs expected, not “consider adding.”
