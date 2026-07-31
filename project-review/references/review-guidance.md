# Review guidance package — build procedure

Canonical procedure for building the **review guidance package** used by `/project-review deep` (and optionally a light package for fast).

Parent skill: [`../SKILL.md`](../SKILL.md)  
Template: [`review-guidance-template.md`](review-guidance-template.md)  
Deep protocol: [`deep-mode.md`](deep-mode.md)  
Schema: [`inventory.schema.json`](inventory.schema.json)

---

## Why this exists

Deep review is multi-agent and long-running. Workers need a single **authority document** so they:

1. Review the same intended state and quality bar
2. Know where to write candidates (never Linear)
3. Apply the same lenses and signal filter
4. Mark coverage even when findings are zero

Without a package, workers invent inconsistent tickets and the orchestrator cannot prove coverage.

---

## When to run

| Mode | Required? |
|------|-----------|
| **Deep** | **Yes** — before any review worker |
| **Fast** | Optional light package; still prefer writing candidates to disk before Linear create |

---

## Scratch paths

Created in deep-mode Phase D1. Absolute paths:

| File | Role |
|------|------|
| `guidance.md` | Human authority for every worker |
| `inventory.json` | All review units |
| `coverage.json` | Unit status + slice assignment |
| `board-snapshot.json` | Offline Linear open-issue keys (Phase D1b) |
| `state.json` | Run metadata |
| `issue-candidates/**` | Candidate lifecycle (see issue-candidates.md) |
| `workers/` | Worker coverage reports |

---

## Step A — Intended state into guidance

From Phase 1 ([`intended-state.md`](intended-state.md)):

- Primary journeys
- Secondary features
- Good reference surfaces (for taste matching)
- Constraints and non-goals
- Confidence + assumptions

Paste a compact snapshot into `guidance.md`. Workers must not invent product scope outside this model without code evidence.

---

## Step B — Inventory → coverage

1. Load `inventory.json` from Phase D0.
2. Initialize each unit:

```json
{
  "unit_id": "apps-web-route-dashboard",
  "status": "pending",
  "slice_id": null,
  "worker_task_id": null,
  "lenses_completed": [],
  "findings_count": 0,
  "notes": ""
}
```

3. Write `coverage.json` with `units[]` + summary counts.

---

## Step C — Slice planning

Cluster units:

| Slice pattern | Example `slice_id` |
|---------------|-------------------|
| App + route prefix | `web-dashboard`, `web-settings` |
| API domain | `web-api-projects`, `web-api-auth` |
| Shared UI package | `pkg-ui`, `pkg-design-system` |
| Auth/middleware | `auth-core` |
| Cross-cutting a11y (after surfaces) | `cross-a11y` |
| Live walk batch | `live-public`, `live-app` |

Rules:

- ~5–25 units per slice (split large route trees).
- Prefer cohesive surfaces so workers see sibling patterns.
- Cross-cutting slices run **after** primary surface slices when possible (reuse candidate themes).
- Assign `slice_id` on each unit in `coverage.json` up front (or on claim).

---

## Step D — Fill guidance.md

Copy [`review-guidance-template.md`](review-guidance-template.md) and fill every `{{PLACEHOLDER}}`.

Must include:

1. Run metadata + absolute scratch paths
2. Repo / intended-state snapshot
3. Reference surfaces for taste
4. Quality bar pointers (issue-template, taste-to-concrete, signal filter)
5. Deep lenses required (`[F]` + `[D]`)
6. **Write rules** for `issue-candidates/`
7. **Forbidden:** Linear, implement, edit app source
8. Candidate frontmatter schema
9. Verification commands from AGENTS.md
10. Slice list summary

---

## Step E — state.json

```json
{
  "run_id": "<RUN_ID>",
  "status": "planning",
  "mode": "deep",
  "concurrency": 4,
  "repo_root": "/abs/path",
  "scratch_dir": "/abs/...",
  "guidance_md": "/abs/.../guidance.md",
  "inventory_json": "/abs/.../inventory.json",
  "coverage_json": "/abs/.../coverage.json",
  "board_snapshot_json": "/abs/.../board-snapshot.json",
  "issue_candidates_dir": "/abs/.../issue-candidates",
  "team": "...",
  "project": "...",
  "live_url": null,
  "scope_only": false,
  "file_mode": "file",
  "workers": [],
  "phase": "D1"
}
```

Update `status` / `phase` as the run advances: `planning` → `snapshot` → `reviewing` → `cleanup` → `filing` → `completed` | `stopped`.

---

## Step F — Gate before workers

Do **not** start Phase D3 until:

- [ ] `guidance.md` exists and is complete
- [ ] `inventory.json` has ≥1 product unit
- [ ] `coverage.json` initialized
- [ ] `issue-candidates/` directory tree exists
- [ ] Board snapshot attempted (success or explicit offline)

---

## Inject into every worker

Workers receive absolute paths only:

```markdown
## Review guidance (hard)
- Read fully: <abs guidance.md>
- Inventory: <abs inventory.json>
- Your units: <list unit_ids>
- Write candidates under: <abs issue-candidates/>
- Coverage report: <abs workers/slice_id.md>
- Do **not** call Linear or edit application source
```

---

## Anti-patterns

- Starting workers before guidance.md exists
- Letting each worker invent its own AC quality bar
- Putting Linear API instructions in worker prompts
- Freezing a tiny inventory that omits routes/APIs in deep mode
- Forgetting to update coverage.json after worker completion
