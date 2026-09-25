---
name: issues
description: >
  Research a multi-item product/code dump in the current repo and file many
  execution-ready `.WCP/issues/` files in one pass so cheaper models can
  implement each leaf from the ticket alone. Decompose into atomic solve-ready
  leaves that may be independent or blocked on another file. Reuses /issue
  execution-ready quality bar (code map, contracts, step plan, file-by-file,
  AC, drift check) with shared investigation + batch duplicate scan. Use when
  the user runs /issues, says "file these issues", "ticket this list", or pastes a multi-bullet brain dump /
  residual backlog. Does not implement code. Prefer /issue for a single one-shot
  ticket. Prefer /project-review when the agent must invent findings without a
  user laundry list.
argument-hint: "[--draft] [--plan-only] [--no-epic] [--epic \"Title\"] [--max N]"
---

# /issues — Research and file many execution-ready WCP issues

Bulk intake skill. The user gives a **multi-item** description (list, residual
backlog, brain dump, several related/unrelated problems). You investigate the
repo **once**, decompose into **atomic solve-ready leaves**, decide which
tickets are connected (or not), then write **multiple** files under
`.WCP/issues/` ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)) — each a
complete **implementation contract** for a cheaper coding model. Do not call Linear. Do not implement
code. Do not open a PR.

**North star (per leaf):** a junior engineer or cheap agent who has never seen
this repo can finish **that one ticket** from its file + a short drift
check — without siblings, and without rediscovering architecture.

| Skill | When |
|-------|------|
| `/issue` | One short description → one ticket |
| **`/issues`** | Many items / one dump → many tickets (this skill) |
| `/start` | Greenfield from next-starter-template: docs + Notion issues database + V1 build |
| `/project-review` | Agent invents findings without a user laundry list (whole project) |
| `/walk` | Agent invents findings from a **live UI walk** (front-facing screens only) |
| `/solve` | Implements already-filed tickets (or hand off to Cursor Auto, etc.) |

Full quality bar: [`../issue/references/execution-ready-bar.md`](../issue/references/execution-ready-bar.md).  
Leaf body: [`../issue/references/issue-body-template.md`](../issue/references/issue-body-template.md).  
Decomposition: [references/decomposition.md](references/decomposition.md).  
Direction conflicts: [`../issue/references/direction-conflict.md`](../issue/references/direction-conflict.md).

## Operating contract

- **Many descriptions → many `.WCP/issues/` files** (atomic leaves). Never cram
  unrelated work into one mega-ticket.
- **Shared research, per-leaf depth**: investigate the repo holistically, then
  still write a **self-contained** execution-ready body on every leaf (cheap
  models often see only one issue).
- **Connected only when real**: use epic / `blockedBy` / `relatedTo` only when
  dependency or theme warrants it. Independent items stay flat and unlinked.
- **Cheap-model ready**: each create leaf must pass the same create gate as
  `/issue` (plan, file map, AC, verify, drift, occupancy, assumptions).
- **Occupancy (WCP)**: prefer leaves with **disjoint primary write paths** so
  `/solve` can lease them in one wave ([`../docs/wcp.md`](../docs/wcp.md),
  [references/decomposition.md](references/decomposition.md)). Sharing a file
  is occupancy, not extra `blockedBy`.
- **Do not ask clarifying questions** unless a safety-critical ambiguity would file wrong work. Prefer assumptions
  in ticket bodies + a short batch plan in chat. **Override:** if the dump
  includes "ask me any questions", "ask me any questions if you have them",
  or "ask probing questions", ask probing questions until unambiguous, wait
  for answers, then file. Do not implement.
- **No git commit, push, or PR. No product code changes.**
- **Secrets**: never put tokens, env values, connection strings, or Doppler
  secrets in issue files (env **names** only).
- **Do not call Linear.** Write files per [`../docs/wcp-queue.md`](../docs/wcp-queue.md), then upsert each Notion row ([`../docs/notion-issues.md`](../docs/notion-issues.md)).
- **Unassigned `open/` only**: do not set `assignee` or `in-progress` while filing. A hard dependency is `blocked/` with `reason: blocked by <id>`. There is no epic file.
- **One current direction**: this dump wins over older unstarted tickets (and leftover dump bullets) that contradict it. Drop or retire the old direction — do not file both.

## Args

| Arg | Meaning |
|-----|---------|
| *(none)* | Full flow: research → plan → file |
| `--draft` | Research + plan + full drafted bodies in chat; **do not** write files or Notion |
| `--plan-only` | Stop after the decomposition table (no full bodies, no files, no Notion) |
| `--no-epic` | Never create a parent epic; file flat leaves only |
| `--epic "Title"` | Force a parent epic with this title for connected leaves |
| `--max N` | Cap leaves filed this run (file highest priority first; list deferred) |

Parse args from the user message; ignore unknown tokens after logging them.

## Trigger phrases

`/issues`, `file these issues`, `break this into tickets`, `ticket this list`,
`create multiple Linear issues`, `file a backlog`, `turn this dump into Linear`,
`residual tickets for…`

---

## Workflow

Follow phases in order. Parallelize reads when possible.

### Phase 0 — Capture the dump

1. Treat the full user message (minus args) as the **source dump**.
2. Split into **raw items** using bullets, numbers, headings, blank lines, or
   clear sentence boundaries. If the dump is prose, extract distinct outcomes.
3. For each raw item, draft a one-line **candidate title** and type
   (`bug` | `feature` | `chore` | `regression` | `tech debt` | `docs`).
4. Infer priority language per item when present; default **normal**. Map words through [`../docs/wcp-queue.md`](../docs/wcp-queue.md).
5. If **zero** distinct items: fall back to single-ticket behavior and tell the
   user to prefer `/issue` next time — still file one high-quality issue.
6. If **`--max N`** and candidates exceed N: rank by priority + foundation
   first; only the top N proceed to file; list deferred titles in the reply.

### Phase 1 — Queue

Same as `/issue`: the board is `.WCP/issues/` ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Do not resolve a team or project. Search `open/`, `in-progress/`, and `blocked/` once for the whole dump. Notion uses this repo's origin URL ([`../docs/notion-issues.md`](../docs/notion-issues.md)). Read repo docs for package ownership. Use those package names in titles.

### Phase 2 — Board snapshot, duplicate, and direction-conflict scan (batch)

**REQUIRED:** follow
[`../issue/references/direction-conflict.md`](../issue/references/direction-conflict.md)
before deep writeups. One snapshot for the whole dump — do not re-crawl per leaf.

Search `open/`, `in-progress/`, and `blocked/`. Skip `done/` and `canceled/`. This dump is the current direction. Do not file Y while leaving an unstarted X implementable if they contradict. `## Supersedes` without a status change is **not** enough.

1. One pass over those folders for every candidate surface (routes, feature names, abandoned approach).
2. Build a short **hit list** (id, title, status) for overlaps **and** contradictions.
3. Per candidate:
   - **Exact/near duplicate** → do **not** create; `duplicate_of` existing id
   - **Related, compatible** → create and name the other id in the body
   - **Full contradiction** with this dump’s direction, unstarted, no live lease → create the new leaf, then **retire** the old file (`canceled` + `reason`)
   - **Intra-batch contradiction** (dump still contains leftover X and new Y) → **drop X** (`action: drop-contradicted`); file Y only
   - Live `in-progress` lease or `in-review` → do **not** cancel; plan `Conflict — needs you`
4. `--draft` / `--plan-only`: show retire/drop in the plan; no status writes until a real file exists.

### Phase 3 — Shared codebase investigation

Goal: every leaf is implementable with **light drift verification** only.
Shared research is fine; **leaf bodies must still be self-contained**.

Investigate areas touched by the dump holistically:

| Area | Capture |
|------|---------|
| Surfaces | Routes, screens, APIs, jobs, CLI |
| Code | Primary files, handlers, schemas, migrations |
| Symbols | Names + ~line ranges per leaf cluster |
| Contracts | Types, props, API shapes, schema fields |
| Data | Tables/fields, owning package |
| Config | Feature flags, env **names** only |
| Patterns | Sibling features to **mirror** (path + symbol) |
| Tests / verify | Package scripts from `AGENTS.md` / package.json |
| Ownership | Which monorepo package owns each leaf |
| Occupancy | Primary write path + symbol per leaf; barrels; sibling path overlap |
| Excerpts | Short anchors for non-obvious logic per cluster |

**How (parallelize):**

1. Grep distinctive strings from the whole dump.
2. **Read** highest-signal files for each cluster of candidates (not grep-only).
3. Trace UI → API → service → DB one level for likely roots.
4. Note package boundaries (`app/`, `services/`, `agents/`).
5. For each likely leaf, jot: primary paths, mirror pattern, contracts, verify cmds.
6. Optional: `git log -n 5 -- path` only when it clarifies regressions.

Read-only. No destructive commands, no long unrelated builds.

### Phase 4 — Decompose & connectivity graph

Read [references/decomposition.md](references/decomposition.md).

For each final leaf:

| Field | Meaning |
|-------|---------|
| `temp_id` | `L1`, `L2`, … stable for this run |
| `title` | Specific, searchable, ≤ ~80 chars |
| `type` / `priority` | As inferred |
| `class` | `foundation` \| `feature` \| `polish` \| `content` \| `a11y` \| `chore` |
| `package` | Owning monorepo path |
| `connectivity` | `independent` \| `related` \| `blocked` \| `epic-child` |
| `blocked_by` | List of `temp_id`s (hard deps only) |
| `related` | List of `temp_id`s or existing `TEAM-n` |
| `duplicate_of` | Existing id if skip create |
| `contradicts` | Existing ids or `temp_id`s this leaf retires / drops |

**Atomicity rules (cheap models):**

- One leaf = one shippable outcome with its own acceptance criteria **and** its
  own step plan / file map.
- Split “fix X and also redesign Y” into two leaves.
- Merge only when two bullets are the same change with the same AC.
- Prefer smaller leaves a cheap model can finish in one session over epic novels.
- If a leaf would need multi-package runtime work without explicit integration AC,
  split by package.

**Connectivity rules:**

| Relation | When | File |
|----------|------|------|
| Independent | No shared hard dependency | `open/`, no `reason` |
| Soft related | Same area, either shippable alone | mention the other id in the body |
| Hard blocked | B’s AC impossible until A lands | dependent in `blocked/` with `reason: blocked by <id>` |
| Initiative | ≥2 leaves share one theme | name it in each leaf body; no epic file |

**Initiative name:**

- When **≥2** leaves share a clear initiative, put that name in each leaf body.
- There is no epic file. `--no-epic` is already the file rule. `--epic "Title"`
  is that shared name, not a parent ticket.

**Filing order** (create sequence):

```text
1. foundation (Urgent→High→Medium→Low)
2. feature / a11y-critical
3. polish / content / chore
```

Within a class, file blockers before blocked leaves so identifiers sort sensibly
when no `blockedBy` graph exists.

Produce an internal **batch plan** (see
[references/batch-plan.md](references/batch-plan.md)). Show the user a compact
table before filing (unless `--draft` / `--plan-only` stop earlier):

```markdown
## Plan (N leaves)
| # | Title | P | Class | Links | Action |
|---|-------|---|-------|-------|--------|
| L1 | … | 2 | foundation | — | create |
| L2 | … | 3 | feature | blockedBy L1 | create |
| L3 | … | 3 | feature | dup 0199 | skip |
| L4 | Old modal … | 3 | feature | contradicted by L1 | drop-contradicted |
| L5 | Settings page … | 2 | feature | retire 0040 | create + retire 0040 |
```

If `--plan-only`: stop here.  
If the plan reveals a dangerous ambiguity (wrong product/package): ask one
question; otherwise proceed to draft + file.

### Phase 5 — Draft every leaf (quality bar)

Each **create** leaf uses the body structure in
[`../issue/references/issue-body-template.md`](../issue/references/issue-body-template.md)
(same bar as `/issue`, plus batch metadata):

1. Implementer contract (this leaf only; honor blockedBy)
2. Occupancy (WCP) — primary write path + symbol; sibling overlap
3. Intensity stamp (`## Intensity` — Band + Why + Proof; [`../docs/intensity.md`](../docs/intensity.md))
4. Summary
5. User report (quote the specific bullet/fragment)
6. Current behavior + evidence
6. Expected behavior
7. Suspected root cause / scope
8. Code map (real paths + symbols)
9. Relevant contracts
10. Code anchors + pattern to mirror
11. Step-by-step implementation plan
12. File-by-file changes
13. Do not touch / out of scope
14. Acceptance criteria (checklist)
15. Test plan
16. Verification (real package commands)
17. Drift check
18. Risks / blockers
19. Platform / stack
20. Related / blockedBy / parent
21. Supersedes (if any)
22. Batch metadata (`temp_id`, `class`, batch name)
23. Assumptions / pre-decided

**Self-contained rule:** do not write “see L1 for the schema” without also
summarizing the schema fields L2 needs. A cheaper model may only receive L2.

**Titles:** problem-focused, area prefix when helpful  
`[Agents] Cost page double-counts kickoff reservations`  
No trailing period; no “Fix bug”.

**Labels:** only existing team labels that clearly fit; omit if unsure.

**Epic body** (when creating): short packaging note from
[references/epic-body-template.md](references/epic-body-template.md).  
Never put the only AC on the epic.

### Phase 5B — Per-leaf create gate (fail closed)

For each leaf marked `create`, **do not file** if any fail:

- [ ] Code map paths exist in the workspace now  
- [ ] Step-by-step plan has ≥2 concrete steps  
- [ ] File-by-file table has ≥1 real edit/create path  
- [ ] AC checklist-testable  
- [ ] Verification uses real package scripts  
- [ ] Drift check ≥3 anchors  
- [ ] Assumptions present when the source bullet was thin  
- [ ] No secrets  
- [ ] `## Intensity` stamp with a valid `Band:`  
- [ ] `## Occupancy (WCP)` primary write path filled (or N/A: no application writes)
- [ ] Body self-contained (no “see sibling” as sole context)  
- [ ] Single package ownership (or explicit integration AC)
- [ ] Direction-conflict search ran; unstarted full contradictions have a retire plan (or drop-contradicted intra-batch)

Failed leaves: keep full draft in the reply (or `--draft` mode), mark action
`blocked-thin` in the summary, continue filing other ready leaves.

### Phase 6 — Write the files

Skip if `--draft` or `--plan-only`. Follow [`../docs/wcp-queue.md`](../docs/wcp-queue.md) and [`../docs/notion-issues.md`](../docs/notion-issues.md). Independent leaves go in `open/`. A hard dependency goes in `blocked/` with `reason: blocked by <id>` after the blocker file exists. Do not create an epic file. Do not assign. Do not call Linear.

#### 6A. Leaves in filing order

For each create leaf that passed the gate, write `.WCP/issues/open/<id>-<slug>.md` or `blocked/` when it has a hard dependency. Frontmatter `status` matches the folder. Empty `assignee`, `lease_expires`, and `commit`.

1. Do **not** set assignee or `in-progress`.
2. On success: record `temp_id → id`. Upsert the Notion row at that status.
3. On failure: keep going; report the failed leaf and keep the drafted body.
4. Skip leaves marked `duplicate_of`.
5. Skip leaves that failed the create gate (`blocked-thin`).
6. Skip leaves marked `drop-contradicted`.

#### 6B. Retire contradicted unstarted issues

After ids exist for created leaves. Follow
[`../issue/references/direction-conflict.md`](../issue/references/direction-conflict.md)
**Retire**. Do not retire if that leaf’s create failed. Do not cancel a live lease or an `in-review` file.

### Phase 7 — Reply to the user

```markdown
**Filed:** N created · K skipped (duplicate) · C drop-contradicted · T thin (not filed) · F failed · D deferred (--max)
**Notion:** <database url> · rows written N | failed ids
**Exec-ready:** each created leaf has plan + file map + AC + verify + drift
**Retired:** [0040](path) — contradicted L1 (canceled)   # omit if none
**Conflict — needs you:** [0055](path) — in-progress / foreign lease   # omit if none

| ID | Title | P | Links |
|----|-------|---|-------|
| [0001](path) | … | high | blocked by 0000 |
| [0002](path) | … | normal | related 0001 |
| — | … | normal | **skipped** duplicate of 0199 |
| — | … | Medium | **thin** — missing <…>; draft in thread |

**Focus packages:** `app`, …
**Handoff:** a cheaper model or `/solve` can execute created leaves from the issue file alone.
```

If `--draft`:

```markdown
**Draft only — not filed**
## Plan
<table>
## L1 — <title>
<full body>
…
```

Then stop. Do not implement.

---

## Quality checklist (before create)

- [ ] Team resolved; project set when identifiable  
- [ ] Batch duplicate + direction-conflict scan done (actionable issues)
- [ ] Unstarted contradicted board issues retired after create (or needs-you)
- [ ] Intra-batch X vs Y dropped the contradicted leaf  
- [ ] Leaves are atomic (one outcome each)  
- [ ] Connectivity not over-blocked (soft vs hard deps)  
- [ ] Filing order: foundation before feature/polish  
- [ ] Every create leaf passed Phase 5B create gate  
- [ ] Occupancy (WCP) primary write path on every create leaf (or N/A)  
- [ ] Every create leaf is self-contained (no sibling-only context)  
- [ ] Every create leaf has real code map paths that exist now  
- [ ] Step plan + file-by-file + do-not-touch present  
- [ ] Acceptance criteria checklist-testable  
- [ ] Test plan present (auto and/or manual)  
- [ ] Verification uses this repo’s real scripts  
- [ ] Drift-check anchors included (≥3)  
- [ ] Assumptions / pre-decided filled when bullets were thin  
- [ ] No secrets  
- [ ] Epic has no sole implementable AC (children do)  
- [ ] Monorepo package ownership correct per leaf  

---

## Anti-patterns

- One mega-issue for a multi-bullet dump  
- Filing “investigate X” with no code map or plan  
- Epic-only ticket with all AC on the parent  
- Leaf bodies that say “see epic / see L1” instead of copying needed contracts  
- `blockedBy` webs so dense nothing is `/solve`-eligible  
- Creating duplicates of open board issues  
- Filing Y while leaving unstarted X implementable when they contradict  
- Filing both leftover X and new Y from the same dump  
- Treating `## Supersedes` / `relatedTo` / chat as the retire step  
- Skipping the conflict search because this is “not a stack migration”  
- Asking team/project when docs already say  
- Implementing fixes under this skill  
- Inventing findings the user never mentioned (that’s `/project-review`)  
- Mixing app vs services ownership in one leaf  
- Filing thin shells “to fill in later” — cheap models will freestyle  
- Shared research notes only in chat, not in each leaf body
- Two leaves with the same primary write path when they could split
- Minting `blockedBy` only because two leaves share a file (that is WCP occupancy)  

---

## Notion failure

1. Say what failed
2. Files already written still count as filed
3. Do not pretend the Notion rows exist
4. Do not call Linear

---

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/issue` | Single ticket, rapid-fire one-liner |
| `/issues` | Multi ticket, shared research, graph optional |
| `/start` | New repo from next-starter-template; Notion issues database for that repo; then nested `/solve` |
| `/project-review` | Agent-invented audit → many tickets |
| `/walk` | Live front-facing UI walk → many tickets |
| `/solve` | Implements filed leaves; expands epics |
| `/prb` | Ships code on `dev` → PR → main |
| Cursor Auto / cheap model | Intended **consumer** of tickets this skill files |
