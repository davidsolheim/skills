# Identify user reply (after Phase 5)

The **first** present is one batch. Flexibility is on the **reply**, not a
shortlist on the first pass.

Parse the user’s follow-up once. Prefer the **most specific** match below.

| User says | Action |
|-----------|--------|
| Clear yes: `yes`, `approve`, `go`, `lgtm`, `do it` (whole batch) | Phase 7 on current `PROPOSED` |
| Clear no: `no`, `reject`, `skip`, `next`, `different` (whole batch) | Phase 6 reject: append every `PROPOSED` id to `REJECTED`, re-run Phase 3–5 |
| Approve subset: `yes, drop 3`, `only 1 and 2`, `approve TEAM-123 and TEAM-125` | `PROPOSED` = those ids in original order. Other current batch ids → `REJECTED`. Phase 7. Do not re-rank. |
| Drop one, no yes/no: `drop 3`, `not TEAM-124` | Remove that id from `PROPOSED` → `REJECTED`. If `PROPOSED` still has ≥1, Phase 7. If empty, re-run Phase 3–5. |
| Swap / pin: `swap 4 for TEAM-88`, `add TEAM-88 instead of 2` | Rebuild `PROPOSED` (still ≤ `MAX_N`). New member must be eligible and not in `REJECTED`. Upgrade the new member. **Re-present once.** Do not solve yet. |
| Theme / area: `checkout only`, `just admin` | Set `AREA_FILTER`, re-run Phase 2 filter + 3–5. Current `PROPOSED` ids are **not** auto-rejected unless the user also said skip/reject. |
| `--pick-only` already set, then yes | Phase 7A pick-only (print ids, **no** claim, **no** `/solve`) |
| `fast` on the reply | Set `FAST_ON_APPROVE`. Still need a yes or subset approve. `FAST_OK` from ranking overlap + guidance. |
| Neither | Ask **once**: approve, reject, or subset. Then stop. |

Issue tokens (`TEAM-123`) and 1-based positions (`1`, `2`, `3`) both count.

Reject never un-upgrades Linear bodies.

If nothing remains after reject: say the eligible board is exhausted for this
session and **stop**.
