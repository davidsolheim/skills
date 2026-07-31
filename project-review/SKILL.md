---
name: project-review
description: >
  Fully agentic project review: walk the current repo and (when available) live
  app, invent findings without a human laundry list, and file atomic
  solve-ready Linear issues under an optional epic so /solve can implement them
  end-to-end. Covers completeness, functional bugs, UI consistency, taste/feel,
  edge cases, responsiveness, accessibility, content, and cross-feature
  consistency. Supports fast (high-signal P0/P1 single-agent) and deep
  (exhaustive multi-agent orchestrator with full inventory coverage, local
  issue-candidates package, then one Linear publish pass). Default files to
  Linear; --draft keeps markdown only. Use when the user runs /project-review,
  says review this project, quality pass, UI audit, bug hunt, find issues for
  solve, audit the app, or wants the agent to discover and queue work for /solve.
argument-hint: "[fast|deep] [--draft|--file] [--p0-p1-only|--include-p2] [--no-epic] [--concurrency N] [--scope-only] [--url URL] [surface…]"
---

# /project-review — Agentic Discovery → Solve-Ready Linear Queue

Walk the **current project**, invent what is missing / broken / inconsistent / off, and file a clean set of **atomic Linear leaves** that **`/solve`** can claim and implement without further clarification.

This skill is **discovery and issue creation only**. It never implements application code, never merges, never pushes, never opens PRs, and never runs `/solve` unless the user explicitly asks in the same turn after handoff.

## Difference from `/issue`

| | `/issue` | `/project-review` |
|--|----------|-------------------|
| Who invents the problems? | **Human** prompt | **Agent** (product + code walk) |
| Volume | One ticket per invocation | Many tickets (+ optional epic) |
| Human input | Required problem statement | Optional bias only (surface, URL, notes) |
| Ticket quality bar | Full implementation-ready body | **Same bar** + review metadata |

Optional human notes, Looms, or “focus on billing” **bias** discovery. They are **not** the issue source. Do **not** wait for a laundry list.

## Modes

| Mode | Trigger | Behavior |
|------|---------|----------|
| **Fast** | `fast`, `--fast`, “quick review”, “fast pass” | High-signal single-agent pass. Primary journeys. Prefer P0/P1. Sample secondary lenses. Local candidates preferred before Linear create. |
| **Deep** | `deep`, `--deep`, or **default** when no mode flag | **Exhaustive multi-agent orchestrator.** Full inventory of routes, UI views, APIs, components, auth, jobs. Guidance package + parallel workers until **every unit is terminal**. Local `issue-candidates/` cleaned offline; Linear only via one board snapshot + one publish pass from `final/`. |

**Parsing:** Prefer the last of `fast`/`--fast` vs `deep`/`--deep`. If neither appears → **`deep`**.

Users who want a tight queue for immediate `/solve` should pass **`fast`**. Deep is for a full quality audit that must not skim.

Mode is fixed for the whole run. Report it in the handoff.

### Deep vs fast (architecture)

| | Fast | Deep |
|--|------|------|
| Execution | Single agent, playbook | Orchestrator + workers ([`references/deep-mode.md`](references/deep-mode.md)) |
| Inventory | Primary journeys | Every route / API / shared UI / nav claim |
| Coverage gate | Soft (high-signal done) | Hard: 100% units terminal |
| Guidance package | Optional light scratch | Required (`guidance.md` + inventory + coverage) |
| Candidates | Prefer local files before create | Required `issue-candidates/` tree |
| Linear | Prefer snapshot + file; avoid thrash | **Only** D1b snapshot + D7 publish from `final/` |
| Concurrency | n/a | Default 4, max 8, `--concurrency N` |

## Operating contract

1. **Agent owns the finding list.** Invent issues from intended state + product/code evidence.
2. **Atomic leaves only.** Many small independently solvable issues. Epics package; `/solve` never implements parent shells.
3. **Solve-ready bar.** Every filed leaf uses [`references/issue-template.md`](references/issue-template.md) — same structural quality as `/issue` (code map, drift check, verification, AC, out of scope).
4. **Taste must be executable.** Convert feel/hierarchy problems via [`references/taste-to-concrete.md`](references/taste-to-concrete.md). No “make it premium” as sole AC.
5. **Local-first candidates.** Discovery writes **files** under a scratch package; cleanup and dedupe happen **on disk**. Linear is a publish step, not the working set ([`references/issue-candidates.md`](references/issue-candidates.md)).
6. **Board-aware offline.** One Linear open-issue snapshot for offline `board_match`; do **not** re-list Linear per candidate ([`references/board-sync.md`](references/board-sync.md)).
7. **Code-pin required** for every filed leaf. No zero-anchor polish tickets.
8. **Queue-shaped for solve.** Foundation-first filing order + `blockedBy` when hard deps exist ([`references/dependency-ordering.md`](references/dependency-ordering.md)).
9. **Default file to Linear.** `--draft` is opt-out. When Linear works, create issues (do not ask every time). Deep files **only** from `issue-candidates/final/`.
10. **Scratch lifecycle.** **Filed in Linear (verified) → delete** the run’s `project-review-<RUN_ID>` temp dir. **Not filed** (`--draft`, Linear fail, partial publish) → **keep** the temp dir and put its absolute path in the handoff. Never delete while intended `final/` bodies are only on disk.
11. **Do not claim work.** Backlog/Todo/Triage only; **unassigned**; no start/completion comments; no In Progress. That is `/solve`’s job.
12. **No implementation.** No app code edits, no `/solve`, no push/PR under this skill.
13. **Secrets.** Never put tokens, env values, connection strings, or Doppler secrets in Linear or candidate files.
14. **Linear MCP.** `search_tool` then `use_tool`. Read schemas first. Literal newlines in markdown descriptions (not `\n` escape sequences). Workers **never** call Linear.

## Invocation

```text
/project-review
/project-review fast
/project-review deep
/project-review fast --draft
/project-review deep --p0-p1-only
/project-review deep --concurrency 6
/project-review deep --scope-only apps/web
/project-review fast --include-p2
/project-review --no-epic
/project-review --url https://preview.example.com
/project-review deep apps/web onboarding
/project-review fast dashboard billing
```

### Args

| Arg | Meaning |
|-----|---------|
| `fast` / `--fast` | High-signal P0/P1 on primary journeys (single-agent) |
| `deep` / `--deep` | Exhaustive multi-agent audit (also the default) |
| `--draft` | Local package only; do **not** create Linear issues |
| `--file` | Explicit file mode (default when Linear is available and `--draft` absent) |
| `--p0-p1-only` | After cleanup, publish only P0/P1 (list unfiled P2 in handoff) |
| `--include-p2` | In **fast** mode, allow well-formed high-leverage P2s to be filed |
| `--no-epic` | Flat leaves; no parent epic |
| `--concurrency N` | Deep only. Max simultaneous review workers (1–8). Default **4**. |
| `--scope-only` | Deep only. Limit inventory to SURFACE bias (default: whole project still inventoried) |
| `--url <URL>` | Live app / preview to inspect |
| free text / paths | Scope bias (surface, package, feature area) |

### Parsing order

1. Extract `--url <URL>` (or `url <URL>`).
2. Extract `--concurrency N` / `concurrency N` (do **not** treat as mode).
3. Extract `--draft` / `--file`, `--p0-p1-only`, `--include-p2`, `--no-epic`, `--scope-only`.
4. Detect `fast`/`--fast` or `deep`/`--deep` (last wins; else **deep**).
5. Remainder = surface / scope bias.

Initialize:

- `MODE` = `fast` | `deep`
- `FILE_MODE` = `draft` if `--draft`, else `file` (attempt Linear)
- `EPIC` = false if `--no-epic`, else true
- `P2_POLICY` = from flags + mode defaults (see filing filters)
- `SURFACE` = scope remainder or “whole project”
- `LIVE_URL` = from arg or inferred later
- `CONCURRENCY` = clamp(N, 1, 8) if deep + flag; else **4** for deep; n/a for fast
- `SCOPE_ONLY` = true if `--scope-only`

---

## Trigger phrases

`/project-review`, `project-review fast`, `project-review deep`, `review this project`, `quality pass`, `UI audit`, `taste pass`, `bug hunt`, `find issues for solve`, `audit the app`, `audit the dashboard`, `queue tickets for solve`, `turn this product into Linear issues`

---

## Workflow

### Phase 0 — Bootstrap

1. Parse invocation (above).
2. Confirm workspace root (git repo when possible). Note dirty tree; **read-only on app code** — do not discard user work. Scratch writes under tmp are allowed.
3. Read root `README.md`, `AGENTS.md` / `CLAUDE.md` (and nested AGENTS if monorepo), design-system / architecture docs if present.
4. Inventory structure: packages/apps, primary routes (deep will expand fully in D0), auth model, design tokens / shared UI, package manager.
5. Resolve Linear **team** and **project** using the same priority order as `/issue` and `/solve`:
   1. `AGENTS.md`, `README.md`, `.linear-project`, `.linear.json` / yaml
   2. Memory / recent commits / branch names with issue prefixes
   3. Linear `list_teams` → `list_projects` (prefer active, non-archived)
   4. If still unresolved → ask **once** with top candidates
6. Detect browser tooling (agent-browser, chrome-devtools). Infer `LIVE_URL` from `--url`, README, deploy docs, or Vercel project notes when obvious.
7. Capture default **verification commands** from `AGENTS.md` / package scripts for ticket bodies later.
8. Set skill paths:
   - `REVIEW_SKILL_DIR` = directory containing this `SKILL.md`
   - Load references under `$REVIEW_SKILL_DIR/references/` as needed per phase
9. **Branch on mode:**
   - If `MODE=deep` → after Phase 1, follow [Deep path](#deep-path-orchestrator) (Phases D0–D8). Do **not** use the fast Phase 2 skim as a substitute for coverage.
   - If `MODE=fast` → [Fast path](#fast-path-single-agent) (Phases 1–8 below).

### Phase 1 — Intended state (agent-built)

Build a short internal model of “done and good” **without waiting for the user**.

Full procedure: [`references/intended-state.md`](references/intended-state.md).

Sources (parallelize):

- Product docs, PRD, SOW, Figma links in-repo
- Open + recent Done Linear issues (features, known gaps) — optional light read; deep will snapshot the board once in D1b
- Route map / nav / feature flags in code
- Marketing or landing copy vs app reality
- Optional user bias (focus area, Loom, notes)

Record:

- Primary user journeys
- Secondary in-scope features
- Good reference UI moments already in the product (for taste matching)
- Constraints (mobile-first, a11y, brand)

**Ask the user only** when product intent is genuinely unknowable and filing would create wrong tickets. Prefer assumptions listed on issues.

---

## Deep path (orchestrator)

**Canonical protocol:** [`references/deep-mode.md`](references/deep-mode.md).  
**Guidance package:** [`references/review-guidance.md`](references/review-guidance.md) + template.  
**Candidates:** [`references/issue-candidates.md`](references/issue-candidates.md).  
**Worker prompt:** [`references/worker-prompt.md`](references/worker-prompt.md).

```text
Phase D0    Full project inventory (every route, API, shared component, nav, …)
Phase D1    Write review guidance package (scratch dir)
Phase D1b   ONE Linear board snapshot → board-snapshot.json
Phase D2    Coverage plan report (non-blocking)
Phase D3    Worker loop → issue-candidates/ files
Phase D4    Coverage gate (all units terminal)
Phase D5    Local cleanup (dedupe, board_match offline, pin, final/)
Phase D6    Dependency graph on final/ only
Phase D7    Linear publish from final/*.md  OR  --draft stop
Phase D8    Handoff (filed ids; scratch deleted if fully filed, else keep path)
```

### Deep non-negotiables

1. **Do not stop early** while inventory units remain `pending` / `in_review` / unretried `failed`.
2. **Coverage means units reviewed**, not “one ticket per unit.” Zero findings on a clean unit is success.
3. **Workers never call Linear** and never edit application source.
4. **Linear only twice:** board snapshot (D1b) + publish (D7). No per-candidate `list_issues`.
5. **Publish only** `issue-candidates/final/*.md` after local cleanup.
6. **Concurrency** ≤ 8; default 4. No worktrees for review workers.
7. On Linear publish failure, the local package remains the recovery path — do not re-run full discovery just to re-file.
8. **Filed → delete scratch; not filed → keep scratch** (verify Linear ids before delete).

After Phase 1, **read `deep-mode.md` fully** and execute D0–D8. Quality gates for finals match [Issue quality rules](#issue-quality-rules-non-negotiable) below. Filing filters and Linear hygiene match Phase 7 / linear-filing.

---

## Fast path (single-agent)

Follow phases in order. Parallelize reads and greps within a phase. Prefer writing candidates under a small scratch `issue-candidates/` folder and filing from cleaned finals (same clean-then-file idea; no multi-worker requirement).

### Phase 2 — Agentic discovery (fast)

**The agent invents candidate findings.** Do not ask the user what is wrong.

Full procedure: [`references/discovery-playbook.md`](references/discovery-playbook.md) (**fast** sections).  
Checklist: [`references/review-checklist.md`](references/review-checklist.md) — `[F]` items; `[D]` only if clear P0/P1 appears.

#### 2A. Product / live walk (when URL or local app is available)

- Primary journeys end-to-end
- Auth-gated vs public paths
- Empty / loading / error paths when reachable
- Responsive samples (core)
- Evidence: routes, UI copy, console/network errors, screenshots if tools allow

If no live URL and no running app: **code-only review**; note coverage limits in handoff.

#### 2B. Codebase walk

- Route/page inventory vs intended features (primary focus)
- Stubs, TODOs, placeholder/lorem in production paths
- Missing empty/loading/error patterns vs sibling surfaces
- Design-system drift on high-visibility surfaces
- Form validation, permissions, mutation feedback gaps
- Clear a11y failures on primary controls
- Dead ends: links to missing routes, half-wired features

#### Lenses (fast)

Primary journeys; high-signal only. See checklist `[F]` tags. Taste/hierarchy findings **must** pass through [`references/taste-to-concrete.md`](references/taste-to-concrete.md).

#### Candidate log (prefer files)

Write each finding to scratch `issue-candidates/` when practical (see [`references/issue-candidates.md`](references/issue-candidates.md) fast section). At minimum keep structured records:

- Working title, lens, surface/route
- Priority guess (P0/P1/P2)
- Evidence (URL, UI text, code path if already known)

**Drop pure nits.** Fast is stricter on P2.

### Phase 3 — Board sync (fast)

Before finalizing drafts:

1. Prefer **one** page/list of open issues → treat as snapshot for the run.
2. Classify each candidate offline against that snapshot: **duplicate** · **related** · **conflict** · **new**.
3. Do not re-query Linear for every candidate.
4. Full procedure: [`references/board-sync.md`](references/board-sync.md).

### Phase 4 — Code pin (mandatory for filed leaves)

For every finding that will become a filed (or draft) issue:

1. Grep/read to locate primary files and symbols.
2. Fill **code map** (path, role, approximate lines).
3. Fill **drift check** anchors (paths, exports, routes).
4. Note package/app ownership and verification commands from Phase 0.
5. Visual/taste: pin route + component + tokens or “match surface X”; run taste conversion.
6. If still unpinnable after a reasonable search: **drop** or demote with explicit Assumptions and UI-only route anchor — **never** file “polish X” with zero anchors.

### Phase 5 — Draft solve-ready issues

Produce full drafts using [`references/issue-template.md`](references/issue-template.md). Prefer writing `issue-candidates/final/*.md`.

**Quality gates (all required):**

- [ ] Atomic (one primary change); compound findings split
- [ ] Current behavior + expected behavior
- [ ] Checklist acceptance criteria (verifiable)
- [ ] Verification section (commands + manual steps)
- [ ] Code map + drift check
- [ ] Out of scope / do not change
- [ ] Taste fully concretized when applicable
- [ ] Title follows conventions
- [ ] Priority P0 / P1 / P2 assigned
- [ ] No secrets
- [ ] Platform / stack filled when stack-sensitive (else `none`)

Reject drafts that fail gates. Prefer fewer strong tickets over many weak ones.

### Phase 6 — Dependency graph & packaging

Full procedure: [`references/dependency-ordering.md`](references/dependency-ordering.md).

1. Cluster by surface / theme.
2. Classify each leaf: `foundation` | `feature` | `polish` | `content` | `a11y`.
3. Add hard dependency edges when B’s AC assumes A is done.
4. Default epic title (if `EPIC`):  
   `Review pass ({mode}) – {Project or Surface} – YYYY-MM`
5. **Filing order:** foundations → features → polish/content/a11y.
6. Plan `blockedBy` for hard deps; do **not** use Blocked *state* for normal chains.

### Phase 7 — File to Linear (default)

Full procedure: [`references/linear-filing.md`](references/linear-filing.md).

#### Filing filters

| Mode / flag | What to file |
|-------------|--------------|
| **Fast** (default) | P0 + P1 only |
| **Fast** + `--include-p2` | P0 + P1 + well-formed high-leverage P2 |
| **Deep** (default) | All well-formed P0/P1/P2 in `final/` |
| **Deep** + `--p0-p1-only` | P0 + P1 only from `final/`; list unfiled P2 in handoff |
| **`--draft`** | Nothing in Linear; full local package |

#### Priority → Linear

| Review | Linear `priority` |
|--------|-------------------|
| P0 data loss / broken core production path | `1` Urgent |
| P0 other / most P1 high-visibility | `2` High |
| Remaining P1 / solid P2 | `3` Medium |
| Minor P2 | `4` Low |

Linear map: `0=None, 1=Urgent, 2=High, 3=Medium, 4=Low`.

#### Create sequence

1. If `FILE_MODE=draft` or Linear unavailable → skip create; emit draft package path; handoff says **not filed**.
2. **Deep:** create only from `issue-candidates/final/*.md`. **Fast:** create from finals or equivalent full drafts after cleanup.
3. Create epic first (unless `--no-epic`) with short body: mode, surface, date, planned leaf count, “leaves are solve targets; do not implement epic shell.”
4. Create leaves in dependency/filing order with full template body:
   - team + project
   - parent = epic when applicable
   - priority from map
   - labels if team labels clearly match — never invent labels
   - state = Backlog / Todo / Triage (team default for new work)
   - **unassigned**
5. Set `relatedTo` / `blockedBy` after ids exist if needed.
6. On create failure: retry once after fixing team/project; if still failing, keep draft bodies on disk / in handoff (**do not delete scratch**).
7. **After verified full publish:** delete the run scratch dir (`rm -rf` only `project-review-<RUN_ID>`). Capture Linear ids/URLs for handoff **before** delete. If draft, partial, or not filed → **keep** scratch and report path.

**Never:** assign to self, set In Progress, post start comments, or mark Done.

### Phase 8 — Handoff

Use [`references/handoff-template.md`](references/handoff-template.md). Always include:

- Mode, team/project, epic link (if any)
- Counts filed vs discovered-not-filed by P0/P1/P2
- Unblocked leaves ready for `/solve` (ordered)
- Dependency chains
- Suggested next command (`/solve`, `/solve 5`, `/solve all`, `/solve all fast`)
- Duplicates skipped (existing ids)
- Coverage limits (no live URL, fast deprioritized surfaces)
- **Scratch:** `deleted after successful Linear file` **or** absolute path if kept (draft / partial / not filed)
- **Deep (when kept):** coverage unit stats + package paths
- In fast mode: note that a deep pass is still available

**Stop.** Do not start `/solve` unless the user asks immediately.

---

## Issue quality rules (non-negotiable)

These exist so tickets survive the `/solve` implement→review loop:

1. Never use vague language as the sole AC (“feel premium”, “improve UX”, “polish the page”).
2. Always describe current state so the implementer does not reverse-engineer the problem.
3. Prefer “match existing pattern on [surface]” over inventing new design values.
4. One primary change per issue; split compounds.
5. Verification steps must be runnable or clearly manual (from `AGENTS.md` when possible).
6. Explicit “do not change” lists prevent scope expansion.
7. Code map + drift check required for filed leaves.
8. Write for another agent: paths, symbols, AC beat vague product prose.

---

## Linear hygiene (this skill)

| Action | Allowed? |
|--------|----------|
| Create epic + leaves | Yes (default), from cleaned finals |
| One board snapshot (list open issues) | Yes (once per run for dedupe keys) |
| Per-candidate Linear search loops | **No** |
| Set priority, labels, project, parent | Yes |
| Set `relatedTo` / `blockedBy` | Yes |
| Assign / In Progress / Done | **No** |
| Start/claim/completion comments | **No** |
| Comment on skipped duplicates | Optional one-line only if it prevents re-filing thrash; prefer handoff only |
| Mass-cancel open issues | **No** |
| Workers calling Linear | **No** |

---

## Anti-patterns

- Waiting for the user to list bugs or paste a feedback dump as the only source
- One mega “Fix everything from the review” ticket
- Leaving taste as subjective feelings without concrete AC
- Filing without code map / drift check
- Creating duplicates of open Linear issues (use board snapshot offline)
- **Re-listing Linear for every candidate** during discovery or cleanup
- **Filing Linear from raw worker `_inbox` dumps** without local cleanup
- **Stopping deep early** with pending inventory units (“enough findings”)
- Claiming deep exhaustive coverage after a single-agent skim
- **Deleting scratch while any intended final is unfiled** (draft, partial, failed publish)
- **Leaving scratch forever after a fully verified Linear publish** (should delete the run dir)
- Assigning, claiming, or setting In Progress
- Implementing fixes inside this skill
- Flooding fast mode with P2 nits
- Filing polish before foundations with no `blockedBy` when B requires A
- Inventing platform migrations without repo/Linear evidence
- Asking whether to file when Linear works and user did not pass `--draft`
- Running `/solve` automatically without being asked
- Dumping raw command transcripts into Linear
- Putting secrets in issue bodies
- Using worktrees for deep review workers (unnecessary; discovery-only)

---

## Failure modes

| Situation | Action |
|-----------|--------|
| Linear auth/tools fail | Complete discovery + local finals; handoff with package path; state **not filed** |
| Team/project unresolved | Ask once with candidates; do not invent a team |
| No live URL / no browser | Code-only review; note limits; deep still inventorizes all code units |
| Greenfield / thin docs | Infer from code + common product baselines; heavy Assumptions on tickets |
| Too many deep findings | Signal filter on disk; `--p0-p1-only` if user asked; never file weak nits |
| Create partially fails | Report which ids succeeded; **keep** scratch; remaining `final/` for re-file |
| Full Linear publish verified | **Delete** run scratch dir; handoff has Linear ids only (no recovery path needed) |
| Worker timeout / fail | Mark slice failed; re-queue incomplete units; do not abandon coverage gate |
| Empty inventory (deep) | Stop and report; do not fake exhaustive review |

---

## Relation to other skills

| Skill | Relationship |
|-------|--------------|
| `/project-review` | **Agentic** multi-issue discovery + Linear queue (fast single-agent or deep orchestrator) |
| `/issue` | **Human-prompted** single issue; **shared ticket quality bar** and Linear resolution order |
| `/solve` | **Downstream consumer.** Picks unblocked leaves, implement→review, merges to local `dev`. Deep’s guidance package is for **review**, not solve batch guidance — but leaves must be solve-ready. |
| `/implement` | Not invoked by this skill |
| GSD / ship skills | Not used; no push/PR/deploy here |

### Pipeline

```text
/project-review fast     →  high-signal leaves (Backlog)
/project-review deep     →  full inventory → workers → local final/ → Linear epic + leaves
         ↓
/solve | /solve N | /solve all [fast]  →  local dev
```

---

## Resources

| File | Role |
|------|------|
| [`references/deep-mode.md`](references/deep-mode.md) | **Deep orchestrator protocol** (D0–D8) |
| [`references/review-guidance.md`](references/review-guidance.md) | Build review guidance package |
| [`references/review-guidance-template.md`](references/review-guidance-template.md) | `guidance.md` template |
| [`references/issue-candidates.md`](references/issue-candidates.md) | Local candidate tree, cleanup, publish rules |
| [`references/inventory.schema.json`](references/inventory.schema.json) | Inventory + coverage schema |
| [`references/worker-prompt.md`](references/worker-prompt.md) | Deep worker prompt template |
| [`references/issue-template.md`](references/issue-template.md) | Mandatory body for every leaf |
| [`references/review-checklist.md`](references/review-checklist.md) | Lens checklist (fast vs deep tags) |
| [`references/taste-to-concrete.md`](references/taste-to-concrete.md) | Feel → executable AC |
| [`references/discovery-playbook.md`](references/discovery-playbook.md) | Fast discovery steps; deep defers to deep-mode |
| [`references/intended-state.md`](references/intended-state.md) | Infer “done and good” without a human list |
| [`references/board-sync.md`](references/board-sync.md) | Snapshot + offline dedupe / relate |
| [`references/dependency-ordering.md`](references/dependency-ordering.md) | Foundation order, blockedBy, epic packaging |
| [`references/linear-filing.md`](references/linear-filing.md) | Publish from final/; Linear policy |
| [`references/handoff-template.md`](references/handoff-template.md) | User-facing end summary |
