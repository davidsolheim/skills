# Discovery Playbook — Agentic Finding Generation

Ordered procedure for **fast** Phase 2. The agent **invents** candidates; the user does not supply the issue list.

Optional human bias (surface, URL, “focus on billing”) narrows scope. It does not replace discovery.

---

## Deep mode: do not use this playbook as the main path

When `MODE=deep`, **stop** and follow:

- [`deep-mode.md`](deep-mode.md) — orchestrator, inventory, workers, coverage gate
- [`review-guidance.md`](review-guidance.md) — guidance package
- [`issue-candidates.md`](issue-candidates.md) — local candidate files

Deep still invents findings (via workers), but discovery is **inventory-complete + multi-agent**, not a single-agent skim of this playbook. Lenses and signal filter from [`review-checklist.md`](review-checklist.md) still apply inside workers.

The steps below are for **fast** mode (and as lens reference text for workers if useful).

---

## Step 1 — Route / surface inventory

From code (parallelize greps/reads):

1. App router / pages / screens (`app/`, `pages/`, `src/routes`, native navigators).
2. Primary nav / sidebar / tab config (what users are told exists).
3. Marketing or docs links into the product.
4. Feature flags that gate routes.

Output: list of surfaces with package ownership and auth requirements.

**Scope bias:** If user named a surface, still skim global nav once, then deep-dive the named area.

**Fast:** Rank top 3–7 primary journeys; deprioritize secondary settings/admin unless broken links point there.

---

## Step 2 — Rank journeys from intended state

Using Phase 1 intended state:

1. List primary happy paths (e.g. sign up → first value → core loop).
2. List secondary paths (settings, billing, invites, exports).
3. Mark each: `must_walk` | `sample` | `skip_this_mode`.

Fast: only `must_walk` + critical samples.

---

## Step 3 — Live product walk (when available)

If `LIVE_URL` or local dev is reachable and browser tooling exists:

1. Walk each `must_walk` journey end-to-end.
2. Capture evidence: route, visible bugs, copy problems, console errors.
3. Try empty states when possible (new account, filtered-to-zero, or code-inferred).
4. Resize or device emulation for core responsive checks (fast: mobile + desktop on primary).
5. Note auth walls you cannot pass; fall back to code for those areas.

If live walk is impossible: skip to Step 4 and mark handoff **code-only**.

Do **not** spend the whole run on screenshots; enough evidence to pin tickets is enough.

---

## Step 4 — Code completeness pass

For each important surface:

1. Entry page/layout → main components → data fetch / server actions → schema if relevant.
2. Look for stubs: TODO/FIXME, placeholder copy, empty handlers, fake data, “coming soon”.
3. Compare nav registration vs real page implementation.
4. Compare intended features (Phase 1) vs what code actually implements.
5. Sibling pattern check: does this list have empty/loading/error like the good list next door?

Record candidates with provisional paths even before full code pin (Phase 4 deepens pins).

---

## Step 5 — Pattern consistency pass

Pick 1–3 **reference quality** surfaces already in the product (from intended state).

For each high-traffic surface under review:

1. Spacing, type, button variants vs reference
2. Modal/drawer motion vs reference
3. Empty/loading treatment vs reference
4. Copy tone vs reference

Every consistency gap must be expressible as “match X” or exact tokens — see `taste-to-concrete.md`.

---

## Step 6 — Lens sweep

Run the mode-appropriate items in `review-checklist.md`:

- Fast: all `[F]` items on primary surfaces; `[D]` only if a clear P0/P1 appears mid-pass

Do not create a ticket per checklist checkbox. Checklist is a **search pattern**, not a filing quota.

---

## Step 7 — Candidate extraction rules

A candidate needs:

1. **Surface** (route/page/flow)
2. **Lens**
3. **Observable problem** (current state)
4. **Direction of fix** (expected state sketch)
5. **Priority guess** P0 / P1 / P2
6. **Evidence** (UI, code, or both)

**Prefer writing each candidate to a local file** under a scratch `issue-candidates/` folder (see [`issue-candidates.md`](issue-candidates.md)) rather than only holding them in chat. Clean locally, then file to Linear.

### Priority heuristics

| Priority | Examples |
|----------|----------|
| **P0** | Core flow broken, data loss, auth hole on primary path, prod placeholder blocking use |
| **P1** | High-visibility missing feature, broken secondary but common path, strong hierarchy/taste failure on first-run surfaces, clear primary a11y failure |
| **P2** | Deeper consistency, secondary empty states, content polish, non-critical a11y, dense taste nits that still pass concrete AC |

### Atomicity

Split if one ticket would need multiple unrelated AC clusters (e.g. “fix checkout API” + “restyle footer”).  
Merge only if one smallest change naturally fixes one user-visible problem.

---

## Step 8 — Signal threshold & pruning

Apply the signal filter at the bottom of `review-checklist.md`.

| Mode | Prune rule |
|------|------------|
| Fast | Cap energy on P2; only keep P2 if extraordinary leverage or user-called-out |
| Deep | Handled in deep-mode D5 cleanup; keep well-formed P2; still drop pure nits |

Prefer **fewer stronger tickets**. A 40-ticket nit flood hurts `/solve` more than it helps.

---

## Step 9 — Candidate log format

Prefer disk (issue-candidates). If ephemeral, keep:

```text
CAND-003
  title: Fix empty state on /projects list
  lens: Edge
  surface: /projects
  priority: P1
  class: feature
  evidence: live empty list shows blank card; ProjectsList.tsx ~L80 no empty branch
  status: keep | drop | duplicate?
```

Statuses move to `keep` only after board match + quality gates for that item.

---

## Parallelization tips (fast)

- Inventory routes + read AGENTS + one board snapshot can overlap with early code reads.
- Live walk and code walk can interleave per surface (see bug in UI → immediately pin code).
- Do not open unbounded Linear queries; one snapshot is enough for offline dedupe.

---

## Anti-patterns in discovery

- Only reading README and inventing generic “add tests” tickets
- Filing without opening the main routes in code or browser
- Treating every TODO as a ticket
- Skipping primary journey because docs look complete
- Asking the user “what should I look at?” as the first step
- Using this playbook alone and claiming a **deep** exhaustive review
- Creating Linear issues mid-discovery before local cleanup
- Searching Linear once per candidate
