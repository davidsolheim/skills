# Identify ranking

How `/identify` scores eligible leaves and cuts a batch (hard cap 4).

Eligibility is **not** defined here. Apply `$SOLVE_SKILL_DIR/references/eligibility.md`
first, then thin S0 in [`guidance.md`](guidance.md), then rank remaining leaves.

---

## Sort keys

After guidance bands (only if a platform conflict exists):

1. **Linear priority** (primary) — more urgent first.
2. **User-facing impact** (secondary) — higher impact first.
3. **Identifier number** (tie-break) — `TEAM-(\d+)` ascending.

Do not sort by “same epic” or “unblocks N tickets”. A mixed batch is correct
when those tickets outrank a theme. Area filter (`AREA_FILTER`) is applied
**before** this sort, not as a rank key.

---

## Priority

Use the issue’s Linear priority. Map:

| Linear | Rank key | Sort |
|--------|----------|------|
| Urgent (`1`) | `P1` | 1 (first) |
| High (`2`) | `P2` | 2 |
| Medium (`3`) or None (`0`) | `P3` | 3 |
| Low (`4`) | `P4` | 4 |

If the tool omits priority, treat as Medium.

---

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

---

## Size cut

`MAX_N` from args, clamped **1–4**. Default `MAX_N = 4` (the cap, not the
target). Never exceed `MAX_N`. Never pad with Low/`U=0` chores.

After sorting `ELIGIBLE` minus `REJECTED` (and applying `PIN_ID` — that leaf
is slot 1 if eligible, then fill remaining by rank):

```text
strong(i) = i is Urgent/High OR U >= 3
weak(i)   = i is Low/None AND U <= 1

if len == 0: no batch
if len == 1: batch = [first]
else:
  batch = first 2
  if not (strong(1) AND strong(2)) and weak(2) and (first is at least Medium or U >= 2):
    batch = first 1     # do not pad a good ticket with a chore
  elif MAX_N >= 3 and 3rd exists and not weak(3rd)
       and 3rd.P == 2nd.P and abs(3rd.U - 2nd.U) <= 1:
    batch = first 3     # 3rd is close
    if MAX_N >= 4 and 4th exists and strong(4th) and strong(3rd) and strong(2nd):
      batch = first 4   # wall of P1/P2 user-facing
```

Default in practice is **2** when the top two are already strong. Take 3 only
when the 3rd is close. Take 4 only for a wall of strong leaves.

If `PIN_ID` is eligible, it occupies **slot 1** even when a higher-rank leaf
exists (user named it). Remaining slots follow this cut on the rest. Overlap
never drops the pin; drop the overlapping non-pin instead.

---

## Overlap cut (after size)

Read primary paths from each proposed body’s code map (or infer from title /
obvious route files). Two leaves **overlap** when they share a path or one path
is the same feature file (same route/module, not merely the same app package).

```text
kept = []
for leaf in batch:
  if leaf overlaps any in kept:
    skip leaf this round (stays eligible; not REJECTED)
    continue
  kept.append(leaf)
while len(kept) < min(original_batch_len, MAX_N) and more ranked eligible remain:
  nxt = next ranked leaf that overlaps none of kept
  if none: break
  kept.append(nxt)
PROPOSED = kept
```

If the **entire** top of the board is one hotspot: cap `PROPOSED` at 2 (or 1
if `MAX_N == 1`), set `FAST_OK = false`, and say so in the proposal.

`FAST_OK` is true only when `FAST_ON_APPROVE` and no pair in `PROPOSED`
overlaps and thin S0 found no platform conflict in this batch.

---

## Why line (required in the proposal)

One clause per ticket:

```text
P{1-4} · U{0-4} · <8–12 words: who feels it>
```

Example: `P2 · U4 · checkout submit 500s on card pay`

When guidance applied: prefix `promote` / `rescope` if relevant.
