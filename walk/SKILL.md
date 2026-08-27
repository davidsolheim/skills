---
name: walk
description: >
  Walk every front-facing UI surface in the live app as a user, debug broken
  interactions, and file every bug, idea, and improvement as solve-ready Linear
  issues. Use when the user runs /walk, says "walk the UI", "click through the
  app", "walk every screen", "front-facing UI audit", "debug the whole UI into
  Linear", or "file UI issues from a live walkthrough". Prefer this over
  /project-review when the job is a live click-through of customer-facing pages
  rather than a full product+code review.
argument-hint: "[--url URL] [--draft|--file] [--signed-out|--signed-in] [--no-epic] [--desktop-only|--mobile-only] [surface…]"
---

# /walk — Live Front-Facing UI Walk → Linear Queue

Act as a user **and** a debugger. Visit **every** front-facing screen, exercise
it, and file **every** actionable bug, idea, and improvement as an atomic
solve-ready Linear leaf.

This skill is **walk + debug + issue creation only**. It never implements
application code, never merges, never pushes, never opens PRs, and never runs
`/solve` unless the user asks in the same turn after handoff.

**North star:** after this run, `/solve` can pick unblocked leaves without
re-walking the product.

## Difference from nearby skills

| | `/walk` | `/project-review` | `/issue` / `/issues` |
|--|---------|-------------------|----------------------|
| Who invents findings? | Agent, **from a live UI walk** | Agent, from product **+ code** (live optional) | **Human** dump |
| Coverage | Every **front-facing** route/view | Whole project (APIs, jobs, completeness) | Named items only |
| Live browser | **Required** (fail the run if none) | Preferred | No |
| Filing filter | **Every** actionable finding (bugs, ideas, improvements) | Fast = P0/P1; deep still drops nits | Only what the user listed |

If the user wants APIs, jobs, schema gaps, or a code-only audit → `/project-review`.
If they already know the tickets → `/issue` or `/issues`.

## Operating contract

1. **Browser-first.** Inventory from code, then **drive the live UI**. A code
   skim or a screenshot gallery is not a walk.
2. **Interact, then debug.** Click, type, submit, navigate. Check console,
   failed network, empty/error/invalid, and a second viewport on primary surfaces.
3. **Every finding.** File bugs, ideas, and improvements that a user would
   notice. Drop only non-actionable observations and board duplicates
   ([`references/finding-kinds.md`](references/finding-kinds.md)).
4. **Atomic solve-ready leaves.** Same `/issue` body + create gate. Taste and
   ideas must be executable (match an existing surface or name exact copy/layout).
5. **Coverage gate.** Every front-facing unit is `walked`, `blocked_auth`,
   `blocked_flag`, `broken_load`, or `not_front`. Do not stop because “enough
   tickets.” Zero findings on a clean screen is success for that unit.
6. **Local-first, then Linear.** Candidates on disk; one board snapshot; one
   publish pass. Reuse `/project-review` board-sync and linear-filing.
7. **Unassigned backlog only.** No In Progress, no assignee, no `/solve` claim.
8. **No product code changes.** Scratch files only.
9. **Secrets.** Never put tokens, env values, connection strings, or Doppler
   secrets in Linear or candidate files.
10. **Linear MCP.** `search_tool` then `use_tool`. Schemas first. Literal
    newlines in markdown. Before every comment: `list_comments` on that issue
    ([`../docs/linear-comments.md`](../docs/linear-comments.md)).

## Invocation

```text
/walk
/walk --url https://preview.example.com
/walk --draft
/walk --signed-out
/walk --signed-in
/walk --no-epic
/walk --desktop-only
/walk --mobile-only
/walk onboarding settings
/walk --url https://app.example.com onboarding
```

### Args

| Arg | Meaning |
|-----|---------|
| *(none)* | Walk all front-facing surfaces; file to Linear |
| `--url <URL>` | Live app / preview to drive (else infer) |
| `--draft` | Local candidates only; do **not** create Linear issues |
| `--file` | Explicit file mode (default when Linear works) |
| `--signed-out` | Public/unauthenticated walk only |
| `--signed-in` | Prefer authenticated app chrome (still walk public entry) |
| `--no-epic` | Flat leaves; no parent epic |
| `--desktop-only` / `--mobile-only` | Skip the other viewport sample |
| free text / paths | Surface bias — still inventory everything; walk named areas first, then the rest |

### Parsing order

1. `--url <URL>` (or `url <URL>`).
2. `--draft` / `--file`.
3. `--signed-out` / `--signed-in` (last of the two wins; default = both if login is possible).
4. `--no-epic`, `--desktop-only`, `--mobile-only`.
5. Remainder = surface bias.

Initialize:

- `FILE_MODE` = `draft` if `--draft`, else `file`
- `EPIC` = false if `--no-epic`, else true
- `LIVE_URL` = from arg or inferred
- `AUTH_MODE` = `signed-out` | `signed-in` | `both` (default `both`)
- `VIEWPORTS` = `desktop+mobile` | `desktop` | `mobile`
- `SURFACE` = remainder or “all front-facing”
- `RUN_ID` = short unique id
- `WALK_SKILL_DIR` = directory containing this `SKILL.md`

### Trigger phrases

`/walk`, `walk the UI`, `click through the app`, `walk every screen`,
`front-facing UI audit`, `debug the whole UI`, `file UI issues from a walkthrough`,
`UI walkthrough into Linear`

---

## Skill paths

```text
WALK_SKILL_DIR = directory containing this SKILL.md
PROTOCOL_MD    = $WALK_SKILL_DIR/references/walk-protocol.md
KINDS_MD       = $WALK_SKILL_DIR/references/finding-kinds.md
META_MD        = $WALK_SKILL_DIR/references/walk-metadata.md
HANDOFF_MD     = $WALK_SKILL_DIR/references/handoff.md
```

Resolve companions (first hit):

| Skill | Candidates |
|-------|------------|
| **issue** | `$WALK_SKILL_DIR/../issue/` → `$HOME/.grok/skills/issue/` |
| **project-review** | `$WALK_SKILL_DIR/../project-review/` → `$HOME/.grok/skills/project-review/` |
| **issues** | `$WALK_SKILL_DIR/../issues/` → `$HOME/.grok/skills/issues/` |

Set:

- `ISSUE_SKILL_DIR` — Phase 1 team/project, body template
- `REVIEW_SKILL_DIR` — `references/board-sync.md`, `linear-filing.md`, `taste-to-concrete.md`, `issue-candidates.md`, `dependency-ordering.md`

If **issue** is missing, still walk and draft; do not pretend Linear was filed
to the `/issue` bar without the template. If **project-review** is missing,
still snapshot once and file, but do not invent a second board-sync protocol.

---

## Workflow

Follow phases in order. Parallelize reads **within** a phase. Do not file
Linear during the walk.

### Phase 0 — Bootstrap

1. Parse invocation. Confirm workspace root. **Read-only on app code.**
2. Read `README.md`, `AGENTS.md` / `CLAUDE.md`, design-system notes if present.
3. Inventory **front-facing** routes/views (protocol § Inventory). Exclude
   `/api/*`, webhooks, server-only, install-chrome unless the user sees it.
4. Resolve Linear **team** and **project** using `/issue` Phase 1. Do not ask
   first. Ask once with candidates only if unresolved.
5. Infer `LIVE_URL`: `--url`, README/deploy docs, then a local preview already
   serving. If the app is down, start it with **this repo’s documented** start
   command (`package.json` scripts, `AGENTS.md`, `startup.sh` if present).
6. Detect browser tools: chrome-devtools MCP, agent-browser, browser-use,
   Playwright (or a repo smoke script if present). **If none can drive pages,
   stop** and say `/walk` cannot run (suggest `/project-review` as code-only).
7. Confirm the app is reachable. Wait until it responds. Leave it running.
8. Capture verification commands from `AGENTS.md` / package scripts for ticket bodies.
9. Create scratch: `$TMPDIR/walk-$RUN_ID/` with `coverage.md`, `issue-candidates/`.
10. Read `PROTOCOL_MD` and `KINDS_MD` before Phase 2.

### Phase 1 — Board snapshot (once)

Follow `$REVIEW_SKILL_DIR/references/board-sync.md` **snapshot only**.

Write `board-snapshot.json` under the scratch dir. Do **not** re-list Linear
per finding.

If Linear is down: empty snapshot; continue; `FILE_MODE` may still be `file`
(publish will fail closed and keep scratch).

### Phase 2 — Live walk (required)

Read and execute [`references/walk-protocol.md`](references/walk-protocol.md).

Coverage log every unit:

```text
status: pending | walked | blocked_auth | blocked_flag | broken_load | not_front
```

Rules:

- Named `SURFACE` bias is walked **first**, then remaining front-facing units.
- Signed-out public routes always, unless `--signed-in` and those routes do not exist.
- Signed-in app chrome when `AUTH_MODE` is `signed-in` or `both` **and** login
  is possible without asking the user for passwords. If login is blocked, mark
  those units `blocked_auth` and keep walking public surfaces.
- Viewport: 1280×800 and 375×812 on primary journeys unless a viewport flag
  restricts it. Secondary screens: one viewport is enough unless the finding is
  responsive.
- **Per surface:** load → interact primary controls → invalid/empty where
  reachable → console + failed network → log findings immediately.
- Screenshot under workspace `screenshots/walk-$RUN_ID/` only as evidence for a
  finding or a broken load — not a gallery of every page.
- Workers (optional, >15 front-facing units): `general-purpose` slices, isolation
  `none`, **never** call Linear, write candidate files only. Orchestrator owns
  coverage gate + publish.

**Do not implement fixes during the walk**, including “quick CSS.” Capture a
finding instead.

### Phase 3 — Classify + code pin

For every captured finding, follow `KINDS_MD` then pin code:

1. Kind: `bug` | `idea` | `improvement`
2. Grep/read the route/component. Fill code map + drift anchors (real paths).
3. Taste / idea: convert via `$REVIEW_SKILL_DIR/references/taste-to-concrete.md`.
   If it cannot become concrete AC, **drop** (not a Linear shell).
4. Offline `board_match` against the Phase 1 snapshot (board-sync classify).
5. Write `issue-candidates/` files (issue-candidates.md). Prefer one file per finding.

Unpinnable after a reasonable search: keep only if the route itself is the
anchor and Assumptions state the implementer must confirm the component.
Never file “polish X” with zero paths.

### Phase 4 — Draft solve-ready leaves

Body = `/issue` template
([`../issue/references/issue-body-template.md`](../issue/references/issue-body-template.md))
plus Walk metadata ([`references/walk-metadata.md`](references/walk-metadata.md)).

Create gate (fail closed):

- [ ] Code map paths exist now
- [ ] Implementation notes name a concrete approach
- [ ] AC checklist-testable
- [ ] Verification uses this repo’s scripts
- [ ] Drift check has ≥3 anchors
- [ ] Assumptions filled when the finding was an idea or thin
- [ ] Walk evidence: route + what you clicked/typed/saw
- [ ] No secrets
- [ ] Duplicate/conflict classified against the board snapshot; skip or note needs-you

Failed gate → `blocked-thin` in handoff, keep draft on disk, do not file that leaf.

Move passing leaves to `issue-candidates/final/`.

### Phase 5 — Package

Follow `$REVIEW_SKILL_DIR/references/dependency-ordering.md`.

Default epic title (if `EPIC`):

`UI walk – {Project or Surface} – YYYY-MM-DD`

Epic is packaging only. Prefer reusing an open “UI walk” epic from the snapshot
when same project and same week.

Filing order: foundation → feature/bug → polish/idea/improvement/a11y/content.

### Phase 6 — File to Linear (default)

Follow `$REVIEW_SKILL_DIR/references/linear-filing.md` with this **filter
override**:

| Mode | What to file |
|------|----------------|
| Default | **All** well-formed `final/` leaves (bugs, ideas, improvements), including Low |
| `--draft` | Nothing in Linear |

Priority map (same as project-review):

| Finding | Linear `priority` |
|---------|-------------------|
| P0 broken core path / data loss | `1` Urgent |
| Other P0 / high-visibility P1 bugs | `2` High |
| Remaining P1 / solid improvements | `3` Medium |
| Ideas and minor improvements | `4` Low |

Create sequence: epic (unless `--no-epic`) → leaves in order → `relatedTo` /
`blockedBy`. Skip snapshot duplicates. If a finding fully contradicts an
unstarted board ticket, note it in the handoff (do not file both).

**Never:** assign, In Progress, Done, start comments, mass-cancel.

Scratch: **filed (verified) → delete** `walk-$RUN_ID`. **Not filed / partial /
`--draft` → keep** and put the absolute path in the handoff.

### Phase 7 — Handoff

Use [`references/handoff.md`](references/handoff.md). Then **stop.**

---

## Issue quality rules

1. User report quotes the **walk observation** (route, click, result), not a
   generic “review the page.”
2. Ideas and improvements still need current vs expected + a plan. “Add dark
   mode someday” without a surface and a match-pattern is not a leaf.
3. One primary change per issue.
4. Runtime proof: surface you actually drove + observed end state
   ([`../docs/prove-it-works.md`](../docs/prove-it-works.md)).
5. Do-not-touch lists stay tight so `/solve` does not restyle the whole app.

---

## Linear hygiene

| Action | Allowed? |
|--------|----------|
| Create epic + leaves from `final/` | Yes (default) |
| One board snapshot | Yes |
| Per-finding Linear search | **No** |
| Assign / In Progress / Done | **No** |
| Walk workers calling Linear | **No** |

---

## Anti-patterns

- Screenshot every route and calling it a walk
- Filing from code TODOs you never opened in the browser
- Stopping after 5 bugs while routes remain `pending`
- One mega “Fix the UI” ticket
- Skipping ideas/improvements because `/project-review` would prune them
- Filing “make it nicer” with no match surface or exact AC
- Re-listing Linear per candidate
- Implementing a “tiny fix” mid-walk
- Asking the user which pages to open (inventory is your job)
- Running `/solve` automatically
- Putting secrets or raw console dumps in Linear
- Claiming exhaustive coverage after walking only the home page

## Red flags — stop and resume the protocol

- About to file without having loaded the route
- About to skip remaining routes because the board already has “enough work”
- About to drop an idea because it is Low
- About to use `/project-review` deep-mode workers as a substitute for clicking
- About to mark `walked` after a 200 HTML response with no interaction

---

## Failure modes

| Situation | Action |
|-----------|--------|
| No browser tool | Stop. Do not fake a walk. Point at `/project-review`. |
| App not reachable and cannot start | Stop. Report what you tried. |
| Login impossible | Walk public; `blocked_auth` the rest; still file public findings |
| Linear down | Finish walk + `final/`; handoff **not filed** + scratch path |
| Team/project unresolved | Ask once; do not invent a team |
| Partial publish | Keep scratch; list created vs remaining finals |
| Production URL | Do not submit real payments, emails, or destructive admin; prefer local/preview |

---

## Relation to other skills

| Skill | Relationship |
|-------|--------------|
| `/walk` | Live front-facing click-through → every UI finding into Linear |
| `/project-review` | Whole-project discovery (code + optional live); not a substitute walk |
| `/issue` | Shared team resolution + execution-ready bar |
| `/issues` | Human multi-item dump; walk **invents** the dump from the UI |
| `/identify` | Picks a batch from the queue this skill fills |
| `/solve` | Implements filed leaves |
| `/tidy` | Optional later hygiene; not this skill |

```text
/walk  →  Linear epic + leaves (Backlog)
         ↓
/identify  →  /solve
```
