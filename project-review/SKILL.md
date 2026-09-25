---
name: project-review
description: >
  Fully agentic project review: walk the current repo and (when available) live
  app, invent findings without a human laundry list, and file atomic
  solve-ready `.WCP/issues/` files so /solve can implement them
  end-to-end. Covers completeness, functional bugs, UI consistency, taste/feel,
  edge cases, responsiveness, accessibility, content, and cross-feature
  consistency. Supports fast (high-signal P0/P1 single-agent) and deep
  (exhaustive multi-agent orchestrator with full inventory coverage, local
  issue-candidates package, then one file and Notion publish). Default writes
  `.WCP/issues/` and Notion; --draft keeps markdown only. Use when the user runs /project-review,
  says review this project, quality pass, UI audit, bug hunt, find issues for
  solve, audit the app, or wants the agent to discover and queue work for /solve.
argument-hint: "[fast|deep] [--draft|--file] [--p0-p1-only|--include-p2] [--no-epic] [--concurrency N] [--scope-only] [--url URL] [surface…]"
---

# /project-review — Agentic Discovery → Solve-Ready Queue

Walk the **current project**, invent what is missing / broken / inconsistent / off, and file a clean set of **atomic `.WCP/issues/` leaves** that **`/solve`** can claim and implement without further clarification.

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
| **Fast** | `fast`, `--fast`, “quick review”, “fast pass” | High-signal single-agent pass. Primary journeys. Prefer P0/P1. Sample secondary lenses. Local candidates, then file `.WCP/issues/` and Notion. |
| **Deep** | `deep`, `--deep`, or **default** when no mode flag | **Exhaustive multi-agent orchestrator.** Full inventory of routes, UI views, APIs, components, auth, jobs. Guidance package + parallel workers until **every unit is terminal**. Local `issue-candidates/` cleaned offline; one read of `.WCP/issues/` and one publish from `final/`. |

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
| Queue | One read of `.WCP/issues/`, then file | **Only** D1b snapshot + D7 publish from `final/` |
| Concurrency | n/a | Default 4, max 8, `--concurrency N` |

## Operating contract

1. **Agent owns the finding list.** Invent issues from intended state + product/code evidence.
2. **Atomic leaves only.** Many small independently solvable issues. Epics package; `/solve` never implements parent shells.
3. **Solve-ready bar.** Every filed leaf uses the `/issue` body
   ([`../issue/references/issue-body-template.md`](../issue/references/issue-body-template.md))
   plus Review metadata in [`references/issue-template.md`](references/issue-template.md).
4. **Taste must be executable.** Convert feel/hierarchy problems via [`references/taste-to-concrete.md`](references/taste-to-concrete.md). No “make it premium” as sole AC.
5. **Local-first candidates.** Discovery writes **files** under a scratch package; cleanup and dedupe happen **on disk**. The queue publish is the last step, not the working set ([`references/issue-candidates.md`](references/issue-candidates.md)).
6. **Board-aware offline.** One read of `.WCP/issues/` for offline
   `board_match` **and** direction-conflict classify; do **not** re-read the queue
   per candidate ([`references/board-sync.md`](references/board-sync.md),
   [`../issue/references/direction-conflict.md`](../issue/references/direction-conflict.md)).
   Retire unstarted contradicted ids at publish only. Agent-invented findings
   do not beat an explicit user ticket of the opposite intent.
7. **Code-pin required** for every filed leaf. No zero-anchor polish tickets.
8. **Queue-shaped for solve.** Foundation-first filing order + `blockedBy` when hard deps exist ([`references/dependency-ordering.md`](references/dependency-ordering.md)).
9. **Default file to `.WCP/issues/` and Notion.** `--draft` is opt-out. Write files per [`../docs/wcp-queue.md`](../docs/wcp-queue.md), then upsert Notion ([`../docs/notion-issues.md`](../docs/notion-issues.md)). Do not call Linear. Deep files **only** from `issue-candidates/final/`.
10. **Scratch lifecycle.** **Filed (issue ids verified) → delete** the run’s `project-review-<RUN_ID>` temp dir. **Not filed** (`--draft`, Notion failure, partial publish) → **keep** the temp dir and put its absolute path in the handoff. Never delete while intended `final/` bodies are only on disk.
11. **Do not claim work.** New leaves stay `open` and unassigned. That is `/solve`’s job.
12. **No implementation.** No app code edits, no `/solve`, no push/PR under this skill.
13. **Secrets.** Never put tokens, env values, connection strings, or Doppler secrets in issue files, Notion properties, or candidate files.
14. **Do not call Linear.** Workers never call it. Publish writes issue files, then Notion.

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
| `--draft` | Local package only; do **not** write `.WCP/issues/` or Notion |
| `--file` | Explicit file mode (default when `--draft` is absent) |
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
- `FILE_MODE` = `draft` if `--draft`, else `file`
- `EPIC` = false if `--no-epic`, else true
- `P2_POLICY` = from flags + mode defaults (see filing filters)
- `SURFACE` = scope remainder or “whole project”
- `LIVE_URL` = from arg or inferred later
- `CONCURRENCY` = clamp(N, 1, 8) if deep + flag; else **4** for deep; n/a for fast
- `SCOPE_ONLY` = true if `--scope-only`

---

## Trigger phrases

`/project-review`, `project-review fast`, `project-review deep`, `review this project`, `quality pass`, `taste pass`, `bug hunt`, `find issues for solve`, `audit the app`, `audit the dashboard`, `queue tickets for solve`, `turn this product into Linear issues`

A **live click-through of every front-facing screen** (bugs + ideas + improvements into Linear) is **`/walk`**, not this skill. Keep this skill for whole-project / code-inclusive review. “UI audit” with no live-walk language still belongs here.

---

## Workflow

### Phase 0 — Bootstrap

1. Parse invocation (above).
2. Confirm workspace root (git repo when possible). Note dirty tree; **read-only on app code** — do not discard user work. Scratch writes under tmp are allowed.
3. Read root `README.md`, `AGENTS.md` / `CLAUDE.md` (and nested AGENTS if monorepo), design-system / architecture docs if present.
4. Inventory structure: packages/apps, primary routes (deep will expand fully in D0), auth model, design tokens / shared UI, package manager.
5. The Notion database is this repo's origin URL ([`../docs/notion-issues.md`](../docs/notion-issues.md)). Do not resolve a Linear team or project.
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
- Open and recent `done/` files (features, known gaps) — optional light read; deep reads `.WCP/issues/` once in D1b
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
Phase D1b   Read `.WCP/issues/` once → board-snapshot.json
Phase D2    Coverage plan report (non-blocking)
Phase D3    Worker loop → issue-candidates/ files
Phase D4    Coverage gate (all units terminal)
Phase D5    Local cleanup (dedupe, board_match + conflict classify offline, pin, final/)
Phase D6    Dependency graph on final/ only
Phase D7    Write `.WCP/issues/` and Notion from final/*.md  OR  --draft stop
Phase D8    Handoff (filed ids; scratch deleted if fully filed, else keep path)
```

### Deep non-negotiables

1. **Do not stop early** while inventory units remain `pending` / `in_review` / unretried `failed`.
2. **Coverage means units reviewed**, not “one ticket per unit.” Zero findings on a clean unit is success.
3. **Workers never call Linear** and never edit application source.
4. **Queue only twice:** read `.WCP/issues/` once (D1b), then write files and Notion rows (D7). No per-candidate search.
5. **Publish only** `issue-candidates/final/*.md` after local cleanup.
6. **Concurrency** ≤ 8; default 4. No worktrees for review workers.
7. On file or Notion failure, the local package remains the recovery path — do not re-run full discovery just to re-file.
8. **Filed → delete scratch; not filed → keep scratch** (verify issue ids before delete).

After Phase 1, **read `deep-mode.md` fully** and execute D0–D8. Quality gates for finals match [Issue quality rules](#issue-quality-rules-non-negotiable) below. Filing filters match Phase 7 / linear-filing.

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

1. Read `.WCP/issues/open/`, `in-progress/`, and `blocked/` once. Skip `done/` and `canceled/`.
2. Classify each candidate offline against that snapshot: **duplicate** · **related** · **conflict** · **new**. Conflicts follow direction-conflict.md (retire canonical-vs-abandoned at publish, or drop the finding).
3. Do not re-read the queue for every candidate.
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

Produce full drafts using the `/issue` body plus
[`references/issue-template.md`](references/issue-template.md) Review metadata.
Prefer writing `issue-candidates/final/*.md`.

**Quality gates (all required):**

- [ ] Atomic (one primary change); compound findings split
- [ ] Current behavior + expected behavior
- [ ] Checklist acceptance criteria (verifiable)
- [ ] Verification section (commands + manual steps)
- [ ] Runtime proof filled when in-scope (drive path; visual reference or n/a)
- [ ] Code map + drift check
- [ ] `## Occupancy (WCP)` primary write path filled (or explicit N/A)
- [ ] Out of scope / do not change
- [ ] Taste fully concretized when applicable
- [ ] Title follows conventions
- [ ] Priority P0 / P1 / P2 assigned
- [ ] No secrets
- [ ] Platform / stack filled when stack-sensitive (else `none`)
- [ ] Direction-conflict classified against the snapshot; retire plan or drop-contradicted when needed

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

### Phase 7 — Write `.WCP/issues/` files (default)

Full procedure: [`references/linear-filing.md`](references/linear-filing.md).

#### Filing filters

| Mode / flag | What to file |
|-------------|--------------|
| **Fast** (default) | P0 + P1 only |
| **Fast** + `--include-p2` | P0 + P1 + well-formed high-leverage P2 |
| **Deep** (default) | All well-formed P0/P1/P2 in `final/` |
| **Deep** + `--p0-p1-only` | P0 + P1 only from `final/`; list unfiled P2 in handoff |
| **`--draft`** | Nothing filed; full local package |

#### Priority

| Review | priority |
|--------|----------|
| P0 data loss / broken core production path | `critical` |
| P0 other / most P1 high-visibility | `high` |
| Remaining P1 / solid P2 | `normal` |
| Minor P2 | `low` |

#### Create sequence

Follow [`references/linear-filing.md`](references/linear-filing.md). Write each leaf under `.WCP/issues/` and upsert Notion. No epic file. Do not assign. Do not set `in-progress` or Notion `done`.

1. `--draft` writes nothing and keeps the scratch package.
2. **Deep:** file only `issue-candidates/final/*.md`. **Fast:** file cleaned finals.
3. A hard dependency is `blocked/` with `reason: blocked by <id>`.
4. Retire contradicted unstarted files (direction-conflict.md). Do not cancel a live lease.
5. After every intended final is filed, delete the run scratch dir. If any final is unfiled, keep the scratch path in the handoff.

### Phase 8 — Handoff

Use [`references/handoff-template.md`](references/handoff-template.md). Always include:

- Mode, team/project, epic link (if any)
- Counts filed vs discovered-not-filed by P0/P1/P2
- Unblocked leaves ready for `/solve` (ordered)
- Dependency chains
- Suggested next command (`/identify`, `/solve`, `/solve 5`, `/solve all`)
- Duplicates skipped (existing ids)
- Retired contradicted unstarted ids; Conflict — needs you; drop-contradicted findings
- Coverage limits (no live URL, fast deprioritized surfaces)
- **Scratch:** `deleted after successful file` **or** absolute path if kept (draft / partial / not filed)
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
5. Verification steps must be runnable or clearly manual (from `AGENTS.md` when possible). In-scope UI/auth/billing/API/schema/shared-helper leaves must fill **Runtime proof** in the issue body ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)) — still `/issue` / `/solve`, no extra slash.
6. Explicit “do not change” lists prevent scope expansion.
7. Code map + drift check required for filed leaves.
8. Write for another agent: paths, symbols, AC beat vague product prose.
9. Fill **Occupancy (WCP)** on every leaf (primary write path). Prefer disjoint paths across the batch ([`../docs/wcp.md`](../docs/wcp.md)).

---

## Queue hygiene (this skill)

| Action | Allowed? |
|--------|----------|
| Write `open/` leaves and Notion rows | Yes (default), from cleaned finals |
| One read of `.WCP/issues/` | Yes (once per run) |
| Per-candidate queue re-reads | **No** |
| Set `priority` on the file | Yes |
| Hard dependency | `blocked/` with `reason: blocked by <id>` |
| Assign / `in-progress` / `done` / Notion `done` | **No** |
| Mass-cancel open issues | **No** |
| Targeted retire of unstarted full contradictions | **Yes** (after create; direction-conflict.md) |
| Calling Linear | **No** |

---

## Anti-patterns

- Waiting for the user to list bugs or paste a feedback dump as the only source
- One mega “Fix everything from the review” ticket
- Leaving taste as subjective feelings without concrete AC
- Filing without code map / drift check
- Creating duplicates of open Linear issues (use board snapshot offline)
- Filing Y while leaving unstarted X implementable when they contradict
- Canceling a user ticket because the review invented the opposite
- Treating `## Supersedes` / chat as the retire step
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
| Notion auth fails | Files already written still count; say which rows failed; do not call Linear |
| Notion database missing | Create it per [`../docs/notion-issues.md`](../docs/notion-issues.md); do not invent a team |
| No live URL / no browser | Code-only review; note limits; deep still inventorizes all code units |
| Greenfield / thin docs | Infer from code + common product baselines; heavy Assumptions on tickets |
| Too many deep findings | Signal filter on disk; `--p0-p1-only` if user asked; never file weak nits |
| Create partially fails | Report which ids succeeded; **keep** scratch; remaining `final/` for re-file |
| Full file publish verified | **Delete** run scratch dir; handoff has issue ids (no recovery path needed) |
| Worker timeout / fail | Mark slice failed; re-queue incomplete units; do not abandon coverage gate |
| Empty inventory (deep) | Stop and report; do not fake exhaustive review |

---

## Relation to other skills

| Skill | Relationship |
|-------|--------------|
| `/project-review` | **Agentic** multi-issue discovery + Linear queue (fast single-agent or deep orchestrator) |
| `/walk` | Live **front-facing UI** click-through; files every bug/idea/improvement. Use when the user wants to drive screens, not audit APIs/jobs. |
| `/issue` | **Human-prompted** single issue; **shared ticket quality bar** and Linear resolution order |
| `/solve` | **Downstream consumer.** Picks unblocked leaves, implement→review, merges to local `dev`. Deep’s guidance package is for **review**, not solve batch guidance — but leaves must be solve-ready. |
| `/implement` | Not invoked by this skill |
| `/prb` `/yeet` | Later ship; not this skill |

### Pipeline

```text
/project-review fast     →  high-signal leaves in `open/`
/project-review deep     →  full inventory → workers → local final/ → `.WCP/issues/` leaves + Notion
         ↓
/identify  (human-approved 2–4)  →  /solve 1 per id
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
| [`../issue/references/execution-ready-bar.md`](../issue/references/execution-ready-bar.md) | Shared quality bar |
| [`../issue/references/issue-body-template.md`](../issue/references/issue-body-template.md) | Canonical leaf body |
| [`references/issue-template.md`](references/issue-template.md) | Review metadata + titles + epic shell |
| [`../issue/references/direction-conflict.md`](../issue/references/direction-conflict.md) | Contradiction search + retire |
| [`references/review-checklist.md`](references/review-checklist.md) | Lens checklist (fast vs deep tags) |
| [`references/taste-to-concrete.md`](references/taste-to-concrete.md) | Feel → executable AC |
| [`references/discovery-playbook.md`](references/discovery-playbook.md) | Fast discovery steps; deep defers to deep-mode |
| [`references/intended-state.md`](references/intended-state.md) | Infer “done and good” without a human list |
| [`references/board-sync.md`](references/board-sync.md) | Snapshot + offline dedupe / relate / conflict classify |
| [`references/dependency-ordering.md`](references/dependency-ordering.md) | Foundation order, blockedBy, epic packaging |
| [`references/linear-filing.md`](references/linear-filing.md) | Publish from final/ into `.WCP/issues/` and Notion |
| [`references/handoff-template.md`](references/handoff-template.md) | User-facing end summary |
