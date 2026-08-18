# Identify ranking

How `/identify` scores eligible leaves and cuts a 2–4 issue batch.

Eligibility is **not** defined here. Apply `/solve` Phase 2B–2E first, then rank
the remaining leaves.

## Sort keys

1. **Linear priority** (primary) — more urgent first.
2. **User-facing impact** (secondary) — higher impact first.
3. **Identifier number** (tie-break) — `TEAM-(\d+)` ascending.

Do not sort by “same epic”, “same surface”, or “unblocks N tickets”. A mixed
batch is correct when those tickets outrank a cluster.

## Priority

Use the issue’s Linear priority. Map:

| Linear | Rank key | Sort |
|--------|----------|------|
| Urgent (`1`) | `P1` | 1 (first) |
| High (`2`) | `P2` | 2 |
| Medium (`3`) or None (`0`) | `P3` | 3 |
| Low (`4`) | `P4` | 4 |

If the tool omits priority, treat as Medium.

## User-facing impact

Score from the title + description + labels + surface (routes, user vs admin).
Do not run a live app walk just to score.

| Score | `U` | Means |
|-------|-----|--------|
| Broken or blocked **primary** user path (auth, pay, checkout, core create/read that is the product) | 4 | Users cannot complete the main job |
| High-visibility **working** surface with a clear defect or gap (home, logged-in home, primary nav, main dashboard) | 3 | Users see it on the default path |
| Secondary user-facing (settings, empty/error on a non-primary flow, edge of a real feature) | 2 | Real users, not the default path |
| Internal / admin / ops / implementer-only | 1 | Staff or agents, not end users |
| Docs, meta, chore, debt with **no** user-visible delta | 0 | Hygiene only |

When unsure between two adjacent scores, pick the **lower** one.

Chore that fixes a broken user path is `U=4` (the path matters, not the label).
A “polish hero” ticket on the marketing home is `U=3`, not `U=4`, unless the
page is unusable.

## Size cut (2–4)

After sorting `ELIGIBLE` minus `REJECTED`:

```text
if len == 0: no batch
if len == 1: batch = [first]
if len == 2: batch = first 2
else:
  batch = first 3
  if a 4th exists AND (its priority is Urgent or High
      OR its U >= 3):
    batch = first 4
  if the 3rd has priority Low AND U <= 1
     AND the 2nd is at least Medium or U >= 2:
    batch = first 2   # do not pad with chores
```

Never more than 4. Never add a ticket with `U=0` and Low/None priority just to
fill a slot when a smaller batch already has Urgent/High or `U>=3` work.

## Why line (required in the proposal)

One clause per ticket:

```text
P{1-4} · U{0-4} · <8–12 words: who feels it>
```

Example: `P2 · U4 · checkout submit 500s on card pay`
