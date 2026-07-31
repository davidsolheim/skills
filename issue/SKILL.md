---
name: issue
description: >
  Investigate a user-described product/code issue in the current repo, gather
  file-level context, and create a detailed Linear issue ready for
  implementation. Auto-resolve Linear team and project from repo context
  (AGENTS.md, CLAUDE.md, agent memory / workspace docs, .linear config, or
  similar project docs)—do not ask the user unless unresolved. Use when the
  user runs /issue, says "create a Linear issue", "file this bug", "log this
  issue", or describes a bug/feature and wants a thorough Linear ticket.
  Optimized for rapid-fire: one description in → deep investigation → one
  complete Linear issue out. Issues should be detailed enough that a later
  agent only needs light verification that files haven't drifted before
  implementing.
---

# /issue — Investigate and File a Linear Issue

Rapid-fire intake skill. The user gives **one** short description. You deeply investigate the current repo, then create **one** implementation-ready Linear issue. Do not implement code. Do not open a PR.

## Operating contract

- **One description → one Linear issue** per invocation unless the user explicitly batches multiple.
- **Speed of interaction, depth of investigation**: keep the user conversation short; put thoroughness in the Linear ticket.
- **Do not ask clarifying questions** unless blocked on Linear team/project resolution or a safety-critical ambiguity that would create a wrong ticket. Prefer stating assumptions in the issue body.
- **No git commit, push, or PR.**
- **No code changes** unless the user explicitly asks for a fix in the same turn (then this skill does not apply).
- **Secrets**: never put tokens, env values, connection strings, or Doppler secrets in Linear.
- **Linear MCP**: discover tools with `search_tool` then call via `use_tool`. Prefer `linear__save_issue` to create. Read tool schemas before calling. Use literal newlines in markdown descriptions (not `\n` escape sequences).

## Trigger phrases

`/issue`, `create a Linear issue`, `file this bug`, `log this issue`, `ticket this`, `create a ticket for…`

---

## Workflow

Follow phases in order. Parallelize reads when possible.

### Phase 0 — Capture the user description

1. Treat the user's message (or `/issue` args) as the sole problem statement.
2. Infer type: **bug**, **feature**, **chore**, **regression**, **tech debt**, or **docs**.
3. Infer rough priority from language (e.g. "broken checkout" → High; "nice to have" → Low). Default **Medium (3)** if unclear.
   - Priority map for Linear: `0=None, 1=Urgent, 2=High, 3=Medium, 4=Low`

### Phase 1 — Resolve Linear team and project (automatic)

**Do not ask the user first.** Resolve from the current workspace using this priority order. Stop at the first high-confidence match; still confirm project exists via Linear list tools before create.

#### 1A. Repo / agent docs (highest priority)

Search and read, in order:

1. Repo root and nested `AGENTS.md`, `Agents.md`, `CLAUDE.md`, `GEMINI.md`
2. `README.md` sections mentioning Linear, team, project, issue prefixes
3. Explicit config files if present:
   - `.linear-project` (single line: project name or slug) — used by related skills
   - `.linear.json`, `.linear.yaml`, `linear.toml`, or similar
4. `docs/` operations or ownership docs that name Linear team/project
5. Package / app metadata (`package.json` name, monorepo workspace names)

Extract: team name/key, project name/slug, issue prefix (e.g. `TEAM`, `ENG` → `PREFIX-N` like `TEAM-123`, `PROJ-42`), default labels, board conventions.

#### 1B. Agent memory and session context

- Agent memory / workspace docs (`MEMORY.md`, project notes, session memory) when available
- Recent Linear issue IDs in git log / branch names (`TEAM-123`, `feat/TEAM-…`, `PREFIX-N`)
- Open PRs or commit messages referencing Linear identifiers

#### 1C. Linear API disambiguation

Using Linear MCP:

1. `linear__list_teams` — match name/key against repo name, product name, or extracted team string
2. `linear__list_projects` with `team` and/or `query` from repo product name
3. Prefer **active / non-archived** projects whose name closely matches the product or monorepo
4. If multiple projects match, prefer:
   - exact product name match
   - project with recent activity related to this repo
   - project already used by recent issues in this team with the same key prefix

#### 1D. When still unresolved

Ask **one** short question listing the top candidates (team + project). Do not invent a team.

Cache resolution mentally for the rest of the session: if the user fires multiple `/issue` descriptions, reuse the same team/project unless they override.

### Phase 2 — Duplicate / overlap / supersession check (quick)

Before deep writeup:

1. `linear__list_issues` with `team`/`project` and a `query` from distinctive error text, route, feature name, or **platform/stack** (Neon, ClickHouse, Convex, etc.)
2. If a clear duplicate exists: **do not create a new issue**. Reply with the existing identifier + URL and note overlap. Offer to add a comment with the new report details if useful.
3. If related but not duplicate: proceed and set `relatedTo` (or note the relation in the body) when creating.
4. If this ticket **changes architectural direction** or **migrates off a stack**:
   - Search open issues that still target the abandoned stack
   - List them under `## Supersedes` (full or partial + override scope)
   - Prefer `relatedTo` those ids; note that `/solve all` will order migration before obsolete features
5. If this ticket is a **feature on a stack that may already be abandoned** in docs/newer tickets: call that out under Risks and link the migration ticket if found.

### Phase 3 — Thorough codebase investigation

Goal: an implementer should need only **light drift verification** (paths still exist, symbols still named similarly) before coding.

Investigate enough to pin:

| Area | What to capture |
|------|-----------------|
| Surface | Routes, screens, API endpoints, CLI commands, jobs |
| Code | Primary files, components, handlers, schemas, migrations |
| Data | Tables/collections, fields, relevant package (`packages/db`, etc.) |
| Config | Feature flags, env var **names** only, provider toggles |
| Patterns | Existing similar features to mirror |
| Tests | Existing test files / commands that should cover the change |
| Docs | Runbooks or acceptance docs already describing desired behavior |

**How to investigate (parallelize):**

1. Grep distinctive strings from the user description (error messages, UI copy, route paths, function names).
2. Read the highest-signal files fully enough to understand current behavior.
3. Trace call chain one level up and down from the suspected root (UI → action/API → service → DB).
4. Note nearby patterns the fix should follow (sibling components, similar endpoints).
5. Check recent git history on those files only if it clarifies regressions (`git log -n 5 -- path`).
6. For UI bugs: note viewport/layout assumptions if obvious from code (mobile vs desktop).
7. For monorepos: state which app/package owns the change (`apps/web`, `apps/native`, `packages/db`, …).

**Do not** run destructive commands, mutate the DB, or start long unrelated builds. Read-only investigation only. Light typecheck/build is optional and usually skipped for intake speed.

### Phase 4 — Draft the issue (quality bar)

#### Title

- Imperative or problem-focused, ≤ ~80 chars when possible
- Include area prefix when helpful: `[Order] Checkout fails when cart has modifiers`
- No trailing period; no vague titles like "Fix bug"

#### Labels

If team labels exist (`linear__list_issue_labels`), apply only labels that clearly fit (e.g. `Bug`, `Web`, `Native`). Do not invent labels. Omit labels when unsure.

#### Priority

Set from Phase 0. User-stated urgency wins.

#### Description (markdown)

Use the structure in [references/issue-body-template.md](references/issue-body-template.md). Every filed issue should include:

1. **Summary** — 2–4 sentences restating the problem/request in product + technical terms
2. **User report** — quoted or paraphrased original description
3. **Current behavior** — what the code/UI does today (with evidence)
4. **Expected behavior** — concrete, testable outcome
5. **Suspected root cause / scope** — hypothesis with file evidence (not a speculative essay)
6. **Code map** — table or list of relevant paths with role (entry point, UI, API, schema, util). Include symbol names and approximate line ranges when known (`apps/web/app/order/page.tsx` ~L40–90)
7. **Implementation notes** — recommended approach, patterns to reuse, pitfalls, out-of-scope items
8. **Acceptance criteria** — checklist of verifiable outcomes
9. **Verification** — exact commands / manual checks the implementer should run (from `AGENTS.md` / README when available)
10. **Drift check** — short list of anchors an agent should re-verify before coding (key files, exports, routes). If these still match, proceed without re-researching the whole area
11. **Risks / blockers** — credentials, migrations, provider limits, related tickets
12. **Platform / stack** — canonical systems this work targets; any stacks it must not use
13. **Supersedes** — when this ticket replaces earlier open or Done work (full vs partial + override scope); omit if none
14. **Assumptions** — anything inferred because the user was brief

Write for another agent: specific paths, symbol names, and acceptance criteria beat vague product prose. Migration tickets should explicitly name abandoned platforms so `/solve all` batch guidance can order work correctly.

### Phase 5 — Create the Linear issue

1. Confirm team (required) and project (when resolved).
2. Call `linear__save_issue` **without** `id` (create mode):

```text
title: <title>
team: <team name or id>
project: <project name, id, or slug>   # when known
description: <full markdown body>
priority: <0-4>
labels: ["Bug", ...]                   # only if confident
relatedTo: ["TEAM-123"]                # optional, related not duplicate
```

3. On success, capture identifier (e.g. `TEAM-123`, `PROJ-42`) and URL.
4. If create fails due to project/team mismatch, fix resolution and retry once. If still failing, report the error and the drafted body so nothing is lost.

### Phase 6 — Reply to the user (keep it short)

Rapid-fire reply format:

```markdown
**Created:** [TEAM-123](url) — <title>
**Team / Project:** <team> / <project>
**Priority:** <level>
**Focus:** <one-line scope, primary paths>
```

If duplicate found instead:

```markdown
**Existing:** [TEAM-123](url) — appears to cover this
**Overlap:** <one sentence>
```

Then stop and wait for the next description. Do not start implementing.

---

## Rapid-fire session mode

When the user sends multiple issues back-to-back:

1. Reuse resolved team/project unless they change repos or say otherwise.
2. Still run investigation + duplicate check per item (do not copy-paste shallow tickets).
3. Do not batch multiple unrelated problems into one Linear issue unless the user asks.
4. Keep each user-facing confirmation to a few lines so the loop stays fast.

---

## Quality checklist (before create)

- [ ] Team resolved from repo docs / agent memory / Linear; project set when identifiable
- [ ] Duplicate check done
- [ ] Code map lists real paths that exist in the workspace right now
- [ ] Acceptance criteria are checklist-testable
- [ ] Verification steps match this repo's real scripts (`AGENTS.md` / package scripts)
- [ ] Drift-check anchors included
- [ ] No secrets in the body
- [ ] Assumptions explicitly listed when the user description was thin
- [ ] Platform/stack noted when the change is stack-sensitive
- [ ] Supersedes filled when this ticket replaces earlier direction or features

---

## Anti-patterns

- Filing "investigate X" with no code map
- Asking the user for team/project when `AGENTS.md` or `.linear-project` already says
- Creating a second issue for an obvious duplicate
- Filing a migration without listing open tickets on the abandoned stack under Supersedes/Related
- Dumping raw command transcripts into Linear
- Implementing the fix under this skill
- Blocking on perfect root cause when a solid scope + file map is enough to start

---

## Linear MCP failure

If Linear auth/tools fail:

1. Say that Linear is unavailable and what failed
2. Still complete investigation
3. Output the full drafted issue (title + body + suggested team/project/priority/labels) in the chat so the user can paste it
4. Do not pretend the issue was created
