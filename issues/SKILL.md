---
name: issues
description: >
  Research a multi-item product/code dump in the current repo and file many
  implementation-ready Linear issues in one pass. Decompose into atomic
  solve-ready leaves that may be independent, related, blockedBy-linked, or
  packaged under an optional epic. Reuses /issue quality bar (code map,
  acceptance criteria, drift check) with shared investigation + batch
  duplicate scan. Use when the user runs /issues, says "file these issues",
  "break this into Linear tickets", "ticket this list", "create multiple
  Linear issues", or pastes a multi-bullet brain dump / residual backlog.
  Does not implement code. Prefer /issue for a single one-shot ticket.
  Prefer /project-review when the agent must invent findings without a user
  laundry list.
argument-hint: "[--draft] [--plan-only] [--no-epic] [--epic \"Title\"] [--max N]"
---

# /issues — Research & File Many Linear Issues

Bulk intake skill. The user gives a **multi-item** description (list, residual
backlog, brain dump, several related/unrelated problems). You investigate the
repo **once**, decompose into **atomic solve-ready leaves**, decide which
tickets are connected (or not), then create **multiple** Linear issues ready
for `/solve`. Do not implement code. Do not open a PR.

| Skill | When |
|-------|------|
| `/issue` | One short description → one ticket |
| **`/issues`** | Many items / one dump → many tickets (this skill) |
| `/project-review` | Agent invents findings without a user laundry list |
| `/solve` | Implements already-filed tickets |

## Operating contract

- **Many descriptions → many Linear issues** (atomic leaves). Never cram
  unrelated work into one mega-ticket.
- **Shared research, per-leaf depth**: investigate the repo holistically, then
  still pin file-level evidence on every leaf.
- **Connected only when real**: use epic / `blockedBy` / `relatedTo` only when
  dependency or theme warrants it. Independent items stay flat and unlinked.
- **Do not ask clarifying questions** unless blocked on Linear team/project or
  a safety-critical ambiguity that would file wrong work. Prefer assumptions
  in ticket bodies + a short batch plan in chat.
- **No git commit, push, or PR. No product code changes.**
- **Secrets**: never put tokens, env values, connection strings, or Doppler
  secrets in Linear.
- **Linear MCP**: `search_tool` then `use_tool`. Prefer `linear__save_issue`
  without `id` to create. Read schemas first. Literal newlines in markdown
  (not `\n` escape sequences).
- **Unassigned backlog only**: do not assign; do not set In Progress/Done.

## Args

| Arg | Meaning |
|-----|---------|
| *(none)* | Full flow: research → plan → file |
| `--draft` | Research + plan + full drafted bodies in chat; **do not** create in Linear |
| `--plan-only` | Stop after the decomposition table (no full bodies, no Linear create) |
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
4. Infer priority language per item when present; default **Medium (3)**.
   Linear map: `0=None, 1=Urgent, 2=High, 3=Medium, 4=Low`.
5. If **zero** distinct items: fall back to single-ticket behavior and tell the
   user to prefer `/issue` next time — still file one high-quality issue.
6. If **`--max N`** and candidates exceed N: rank by priority + foundation
   first; only the top N proceed to file; list deferred titles in the reply.

### Phase 1 — Resolve Linear team and project (once)

Same priority order as `/issue` / `/solve`. Do **not** ask first.

1. Repo docs: `AGENTS.md`, `CLAUDE.md`, `.linear-project`, `.linear.json` / yaml, README Linear notes
2. Memory / recent commits / branch names with issue prefixes (`PREFIX-N`, e.g. `TEAM-123`, `ENG-12`)
3. Linear: `list_teams` → `list_projects` for that team; prefer an active project
   matching the product/repo name from docs; skip archived/completed umbrellas
4. Prefer monorepo package/app language in titles from package ownership docs
   (e.g. `apps/web`, `packages/api`) when the repo is a monorepo
5. If still unresolved: one short question with top candidates

Reuse team/project for the whole batch.

### Phase 2 — Board snapshot & duplicate scan (batch)

Before deep writeups:

1. `linear__list_issues` for the team/project with queries from distinctive
   terms across **all** candidates (routes, feature names, error strings).
2. Build a short **board hit list** (id, title, state) for overlaps.
3. Per candidate later:
   - **Exact/near duplicate** → do **not** create; link existing id in the plan
   - **Related** → create and set `relatedTo`
   - **Supersedes** older open work → document under Supersedes; prefer
     `relatedTo` those ids
4. One snapshot is enough — do not re-crawl the full board per leaf during file.

### Phase 3 — Shared codebase investigation

Goal: every leaf is implementable with **light drift verification** only.

Investigate areas touched by the dump holistically:

| Area | Capture |
|------|---------|
| Surfaces | Routes, screens, APIs, jobs, CLI |
| Code | Primary files, handlers, schemas, migrations |
| Data | Tables/fields, owning package |
| Config | Feature flags, env **names** only |
| Patterns | Sibling features to mirror |
| Tests / verify | Package scripts from `AGENTS.md` / package.json |
| Ownership | Which monorepo package/app owns each leaf |

**How (parallelize):**

1. Grep distinctive strings from the whole dump.
2. Read highest-signal files for each cluster of candidates.
3. Trace UI → API → service → DB one level for likely roots.
4. Note package boundaries (e.g. `apps/web/`, `packages/api/`, `packages/db/`).
5. Optional: `git log -n 5 -- path` only when it clarifies regressions.

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
| `package` | Owning monorepo path (e.g. `apps/web`, `packages/api`) |
| `connectivity` | `independent` \| `related` \| `blocked` \| `epic-child` |
| `blocked_by` | List of `temp_id`s (hard deps only) |
| `related` | List of `temp_id`s or existing `TEAM-n` / `PREFIX-n` |
| `duplicate_of` | Existing id if skip create |

**Atomicity rules:**

- One leaf = one shippable outcome with its own acceptance criteria.
- Split “fix X and also redesign Y” into two leaves.
- Merge only when two bullets are the same change with the same AC.
- Prefer smaller leaves `/solve` can drain over epic novels.

**Connectivity rules:**

| Relation | When | Linear |
|----------|------|--------|
| Independent | No shared hard dependency | No parent, no `blockedBy` |
| Soft related | Same area, either shippable alone | `relatedTo` after create |
| Hard blocked | B’s AC impossible until A lands | `blockedBy` after create |
| Epic cluster | ≥2 leaves share one theme **and** user dump is one initiative | Parent epic + children |

**Epic decision (default):**

- Create an epic when **≥2** leaves share a clear initiative name **and** are
  not pure independent chores from different domains.
- Skip epic when items are a grab-bag of unrelated residuals (`--no-epic`
  always skips).
- `--epic "Title"` forces one parent for all filed leaves this run.
- Epic body is packaging only — **never** the only implementable unit.
  `/solve` expands epics to children.

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
| L3 | … | 3 | feature | dup TEAM-199 | skip |
```

If `--plan-only`: stop here.  
If the plan reveals a dangerous ambiguity (wrong product/package): ask one
question; otherwise proceed to draft + file.

### Phase 5 — Draft every leaf (quality bar)

Each **create** leaf uses the body structure in
[references/issue-body-template.md](references/issue-body-template.md)
(same bar as `/issue`):

1. Summary  
2. User report (quote the specific bullet/fragment)  
3. Current behavior + evidence  
4. Expected behavior  
5. Suspected root cause / scope  
6. Code map (real paths)  
7. Implementation notes  
8. Acceptance criteria (checklist)  
9. Verification (real package commands)  
10. Drift check  
11. Risks / blockers  
12. Platform / stack  
13. Related / blockedBy / parent (temp ids ok pre-file; replace with real ids in Linear relations)  
14. Supersedes (if any)  
15. Assumptions  
16. Batch metadata (optional): `temp_id`, `class`, sibling plan title  

**Titles:** problem-focused, area prefix when helpful  
`[web] Settings page double-counts active seats`  
No trailing period; no “Fix bug”.

**Labels:** only existing team labels that clearly fit; omit if unsure.

**Epic body** (when creating): short packaging note from
[references/epic-body-template.md](references/epic-body-template.md).

### Phase 6 — File in Linear

Skip if `--draft` or `--plan-only`.

#### 6A. Epic (optional)

1. Create parent when Phase 4 says so.
2. Capture `id`, `identifier`, `url`.

#### 6B. Leaves in filing order

For each create leaf:

```text
title: <title>
team: <team>
project: <project when known>
description: <full markdown>
priority: <0-4>
labels: [...]                 # only if confident
parentId / parent: <epic>     # when epic-child
```

3. Do **not** set assignee or In Progress.  
4. On success: record `temp_id → identifier, url, id`.  
5. On failure: keep going; report failed leaf + keep drafted body in the reply.  
6. Skip leaves marked `duplicate_of` (mention existing id instead).

#### 6C. Relations (after ids exist)

1. Map `blocked_by` temp ids → Linear identifiers; set `blockedBy`.  
2. Set `relatedTo` for soft links and board relatives.  
3. If relation API fails: leaves remain valid; note missing links in the reply.

#### 6D. Epic rollup (optional)

Update epic description with child identifiers when easy.

### Phase 7 — Reply to the user

```markdown
**Filed:** N created · K skipped (duplicate) · F failed · D deferred (--max)
**Team / Project:** <team> / <project>
**Epic:** [TEAM-…](url) — <title>   # or “none (flat)”

| ID | Title | P | Links |
|----|-------|---|-------|
| [TEAM-1](url) | … | High | blockedBy TEAM-0 |
| [TEAM-2](url) | … | Medium | related TEAM-1 |
| — | … | Medium | **skipped** duplicate of TEAM-199 |

**Focus packages:** `apps/web`, `packages/api`, …
**Next:** `/solve` or `/solve N` to implement; `/issues --draft` to re-plan.
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
- [ ] Batch duplicate scan done  
- [ ] Leaves are atomic (one outcome each)  
- [ ] Connectivity not over-blocked (soft vs hard deps)  
- [ ] Filing order: foundation before feature/polish  
- [ ] Every create leaf has real code map paths that exist now  
- [ ] Acceptance criteria checklist-testable  
- [ ] Verification uses this repo’s real scripts  
- [ ] Drift-check anchors included  
- [ ] No secrets  
- [ ] Epic has no sole implementable AC (children do)  
- [ ] Monorepo package/app ownership correct per leaf  

---

## Anti-patterns

- One mega-issue for a multi-bullet dump  
- Filing “investigate X” with no code map  
- Epic-only ticket with all AC on the parent  
- `blockedBy` webs so dense nothing is `/solve`-eligible  
- Creating duplicates of open board issues  
- Asking team/project when docs already say  
- Implementing fixes under this skill  
- Inventing findings the user never mentioned (that’s `/project-review`)  
- Mixing two packages’ runtime ownership in one leaf without an explicit integration AC  

---

## Linear MCP failure

1. Say Linear is unavailable and what failed  
2. Still finish research + plan + full drafted bodies  
3. Output the batch in chat for paste  
4. Do not pretend issues were created  

---

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/issue` | Single ticket, rapid-fire one-liner |
| `/issues` | Multi ticket, shared research, graph optional |
| `/project-review` | Agent-invented audit → many tickets |
| `/solve` | Implements filed leaves; expands epics |
| `/prb` | Ships code on `dev` → PR → main |
