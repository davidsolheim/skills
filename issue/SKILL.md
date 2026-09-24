---
name: issue
description: >
  Investigate a user-described product/code issue in the current repo, gather
  deep file-level context, and write one execution-ready issue under
  `.WCP/issues/open/` so a cheaper coding model can implement from that file
  alone with only light drift verification. Do not call Linear. Use when the
  user runs /issue, says "file this bug", "log this issue", or describes a
  bug/feature and wants a thorough ticket. Optimized for: Grok researches
  and writes the contract; a cheaper model executes later. Prefer /issues for
  multi-item dumps.
---

# /issue — Investigate and file one execution-ready WCP issue

Rapid-fire intake skill. The user gives **one** short description. You deeply
investigate the current repo, then write **one** file in `.WCP/issues/open/`
([`../docs/wcp-queue.md`](../docs/wcp-queue.md)) that is a complete
**implementation contract** for a cheaper model. Do not call Linear. Do not implement code. Do not
open a PR.

**North star:** a junior engineer or cheap agent who has never seen this repo can
finish the ticket from that file + a short drift check — without rediscovering
architecture or inventing a new design.

Full quality bar: [references/execution-ready-bar.md](references/execution-ready-bar.md).  
Body structure: [references/issue-body-template.md](references/issue-body-template.md).  
Direction conflicts: [references/direction-conflict.md](references/direction-conflict.md).  
Intensity stamp: [`../docs/intensity.md`](../docs/intensity.md) — `/solve` and `/prb` auto-dial from `## Intensity`.

## Operating contract

- **One description → one `.WCP/issues/` file** per invocation unless the user explicitly batches multiple.
- **Speed of interaction, depth of ticket**: keep the user conversation short; put thoroughness in the issue file.
- **Cheap-model ready**: every filed issue must include code map, contracts, step-by-step plan, file-by-file changes, AC, verification, drift check, Occupancy (WCP), and pre-decided assumptions. Thin tickets fail the create gate.
- **Do not ask clarifying questions** unless a safety-critical ambiguity would create a wrong ticket. Prefer stating assumptions in the issue body. There is no team or project to resolve.
- **No git commit, push, or PR.**
- **No code changes** unless the user explicitly asks for a fix in the same turn (then this skill does not apply).
- **Secrets**: never put tokens, env values, connection strings, or Doppler secrets in the issue file (env **names** only).
- **One current direction**: this invocation wins over older open tickets that contradict it. Cancel those with `reason` after the new file exists. Do not cancel a live `in-progress` lease.
- **Do not call Linear.** Write the file per [`../docs/wcp-queue.md`](../docs/wcp-queue.md).

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
4. If the description clearly contains **multiple independent outcomes**, say so and prefer `/issues` — or split only if the user insists on one ticket (then warn that cheap-model execution will suffer).

### Phase 1 — Queue

The queue is `.WCP/issues/` in this checkout ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Do not resolve a team or project. Do not ask where to file. Search `open/`, `in-progress/`, and `blocked/` for duplicates before writing.

### Phase 2 — Duplicate / overlap / direction-conflict check

**REQUIRED:** follow [references/direction-conflict.md](references/direction-conflict.md) before the deep writeup. Search **non-implemented** issues (Backlog / Todo / unstarted / started — never an unfiltered Done dump), not title-duplicates only.

This invocation is the current direction. If older unstarted tickets still specify the opposite approach (do X vs do Y, keep vs remove, stack A vs B, modal vs page), **retire** them after create. `## Supersedes` + `relatedTo` without a status change is **not** enough — `/solve 1` on the old ticket will still implement it.

1. Duplicate → **do not create**. Reply with the existing identifier + URL.
2. Related, compatible → create and set `relatedTo`.
3. Full contradiction, unstarted, high confidence → create this ticket, then Canceled or Duplicate + comment on the old ids.
4. In Progress (foreign claim) or In Review → do **not** cancel; list **Conflict — needs you**.
5. Ambiguous *new* directions (this message does not pick) → ask once; do not file.

### Phase 3 — Thorough codebase investigation

Goal: the implementer needs only **light drift verification** before coding.
You (Grok) do the expensive research now.

Investigate enough to pin everything in the [execution-ready bar](references/execution-ready-bar.md):

| Area | What to capture |
|------|-----------------|
| Surface | Routes, screens, API endpoints, CLI commands, jobs |
| Code | Primary files, components, handlers, schemas, migrations |
| Symbols | Function/component/type names, ~line ranges |
| Contracts | Props, types, request/response shapes, schema fields |
| Data | Tables/collections, fields, owning package |
| Config | Feature flags, env var **names** only, provider toggles |
| Patterns | Existing similar features to **mirror** (path + symbol) |
| Tests | Existing test files / commands that should cover the change |
| Docs | Runbooks or acceptance docs already describing desired behavior |
| Verify cmds | Exact scripts from `AGENTS.md` / package.json for touched package |

**How to investigate (parallelize):**

1. Grep distinctive strings from the user description (error messages, UI copy, route paths, function names).
2. **Read** the highest-signal files fully enough to understand current behavior (not grep-only).
3. Trace call chain one level up and down from the suspected root (UI → action/API → service → DB).
4. Note nearby patterns the fix should follow (sibling components, similar endpoints) — capture path + symbol for the mirror.
5. Extract **contracts**: types/interfaces, Zod schemas, GraphQL/REST shapes, column names.
6. Capture **short excerpts** (5–40 lines) of non-obvious branches the plan hinges on.
7. Draft an **ordered implementation plan** and **file-by-file change list** while reading (do not leave “how” to the implementer).
8. Resolve package verification commands from `AGENTS.md` / package scripts.
9. Check recent git history on those files only if it clarifies regressions (`git log -n 5 -- path`).
10. For UI bugs: note viewport/layout assumptions if obvious from code (mobile vs desktop).
11. For monorepos: state which app/package owns the change (`apps/web`, `apps/native`, `packages/db`, …).

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

Use the full structure in [references/issue-body-template.md](references/issue-body-template.md). Every filed issue **must** include:

1. **Implementer contract** — scope lock; follow the plan; drift-then-implement
2. **Occupancy (WCP)** — primary write path + symbol; disjoint vs sibling overlap ([`../docs/wcp.md`](../docs/wcp.md))
3. **Intensity** — `## Intensity` with `Band:` `light|standard|heavy|critical`, one-line Why, Proof `on|n/a` ([`../docs/intensity.md`](../docs/intensity.md)). Classify after research; fail closed (bump up when unsure). Do not key off ticket length or priority alone.
4. **Summary** — 2–4 sentences, product + technical
5. **User report** — quoted or paraphrased original description
5. **Current behavior** — what the code/UI does today (with path/symbol evidence)
6. **Expected behavior** — concrete, testable outcomes
7. **Suspected root cause / scope** — hypothesis with file evidence
8. **Code map** — paths + roles + symbols + ~lines; primary package
9. **Relevant contracts** — types, APIs, data, env names, auth/tenancy
10. **Code anchors + pattern to mirror** — short excerpts when non-obvious; always a mirror when one exists
11. **Step-by-step implementation plan** — ordered, mandatory approach unless drift blocks
12. **File-by-file changes** — edit/create/test rows with specific deltas
13. **Do not touch / out of scope** — hard boundaries for cheap models
14. **Acceptance criteria** — outsider-pass/fail checklist
15. **Test plan** — automated cases and/or airtight manual steps
16. **Verification** — exact repo commands + manual pointer
17. **Drift check** — 3–7 anchors + investigation snapshot date
18. **Risks / blockers**
19. **Platform / stack** — canonical vs abandoned systems
20. **Related / Supersedes**
21. **Assumptions / pre-decided** — so the implementer does not guess

Write for another agent that is **less capable than you**. Specific paths, symbols, ordered steps, and AC beat vague product prose. Prefer complete tickets (~80–250 lines body) over short ones missing the how.

### Phase 5 — Create gate, then write the file

#### 5A. Create gate (fail closed)

**Do not call create** if any of these fail:

- [ ] Code map lists real paths that exist in the workspace right now
- [ ] Step-by-step plan has ≥2 concrete steps
- [ ] File-by-file table has ≥1 real edit/create path
- [ ] Acceptance criteria are checklist-testable (not “improve UX”)
- [ ] Verification lists real commands from this repo
- [ ] Drift-check has ≥3 anchors
- [ ] Assumptions filled when the user description was thin
- [ ] No secrets in the body
- [ ] Title is specific
- [ ] `## Intensity` stamp present with a valid `Band:` (`light` `standard` `heavy` `critical`)
- [ ] `## Occupancy (WCP)` primary write path filled (or explicit N/A: no application writes)
- [ ] Direction-conflict search ran (actionable states + surface queries)
- [ ] Unstarted full contradictions have a retire plan (Canceled/Duplicate after create), or the ticket is not filed

If the gate fails: investigate more, or paste the draft in chat and say what is still missing. Do not file a shell ticket.

#### 5B. Write the file

1. Next id per [`../docs/wcp-queue.md`](../docs/wcp-queue.md).
2. Write `.WCP/issues/open/<id>-<slug>.md` with frontmatter `status: open`, empty `assignee`, `lease_expires`, `commit`, and `reason`, plus `priority`, `scope`, `acceptance`, `files: []`, and `created`.
3. The body is the execution-ready contract from Phase 4.
4. Do not assign. Do not set `in-progress`. Do not commit product code. Leave the file in the worktree.
5. If the write fails, report the error and paste the body.

#### 5C. Retire contradicted unstarted issues

Only after create succeeds. Follow [direction-conflict.md](references/direction-conflict.md) **Retire**. Do not retire if create failed. Do not cancel live foreign claims or In Review.

### Phase 6 — Reply to the user (keep it short)

Rapid-fire reply format:

```markdown
**Created:** [TEAM-123](url) — <title>
**Team / Project:** <team> / <project>
**Priority:** <level>
**Intensity:** <light|standard|heavy|critical> (effort <1|2|3|5>)
**Focus:** <one-line scope, primary paths>
**Exec-ready:** plan + file map + AC + verify + drift ✓
**Retired:** [TEAM-40](url) — contradicted this direction (Canceled)   # omit if none
**Conflict — needs you:** [TEAM-55](url) — In Progress / foreign claim   # omit if none
```

If duplicate found instead:

```markdown
**Existing:** [TEAM-123](url) — appears to cover this
**Overlap:** <one sentence>
```

If create gate failed:

```markdown
**Not filed** — ticket not execution-ready yet
**Missing:** <e.g. code map / verification commands>
**Draft:** <paste full body or offer to continue research>
```

Then stop and wait for the next description. Do not start implementing.

---

## Rapid-fire session mode

When the user sends multiple issues back-to-back:

1. Reuse resolved team/project unless they change repos or say otherwise.
2. Still run full investigation + duplicate/direction-conflict check + create gate per item (do not copy-paste shallow tickets).
3. Do not batch multiple unrelated problems into one Linear issue unless the user asks.
4. Keep each user-facing confirmation to a few lines so the loop stays fast.
5. If volume is high and items are multi-bullet, suggest `/issues` for shared research.

---

## Quality checklist (before create)

- [ ] Team resolved from repo/memory/Linear; project set when identifiable
- [ ] Duplicate + direction-conflict check done (actionable issues, not Done dump)
- [ ] Unstarted contradicted issues retired after create (or needs-you if claimed / In Review)
- [ ] Create gate (Phase 5A) passed
- [ ] Intensity stamp valid (`## Intensity` / `Band:`)
- [ ] Occupancy (WCP) primary write path filled (or N/A)
- [ ] Code map lists real paths that exist in the workspace right now
- [ ] Contracts / mirror pattern captured when applicable
- [ ] Step-by-step plan + file-by-file changes present
- [ ] Do-not-touch / out of scope stated
- [ ] Acceptance criteria are checklist-testable
- [ ] Test plan (auto and/or manual) present
- [ ] Verification steps match this repo's real scripts (`AGENTS.md` / package scripts)
- [ ] Drift-check anchors included (≥3)
- [ ] No secrets in the body
- [ ] Assumptions / pre-decided explicitly listed when the user description was thin
- [ ] Platform/stack noted when the change is stack-sensitive
- [ ] Supersedes filled when this ticket replaces earlier direction or features

---

## Anti-patterns

- Filing "investigate X" with no code map or plan
- Filing product prose without file-by-file changes (cheap models will invent scope)
- Asking the user for team/project when `AGENTS.md` or `.linear-project` already says
- Creating a second issue for an obvious duplicate
- Filing Y while leaving unstarted X implementable when X and Y contradict
- Treating `## Supersedes` / `relatedTo` / a chat mention as the retire step
- Skipping the conflict search because this is “not a stack migration”
- Asking “does Y replace X?” when the user just stated Y
- Canceling In Progress (foreign claim) or In Review without asking
- Dumping raw command transcripts or whole files into Linear
- Implementing the fix under this skill
- Blocking on perfect root cause when a solid scope + file map + plan is enough to start
- Mega-tickets that should have been `/issues` splits
- “See related ticket for context” as a substitute for a self-contained body
- Filing without `## Intensity` / `Band:`
- Filing without `## Occupancy (WCP)` (or explicit N/A)
- Classifying intensity from ticket length or Linear priority alone

---

## Relation to other skills

| Skill | Difference |
|-------|------------|
| `/issue` | One ticket on an **existing** repo |
| `/issues` | Many tickets; no implement |
| `/start` | New repo from next-starter-template; new Linear **project** + V1 epic; then build |
| `/solve` | Implements filed leaves |

---

## File write failure

If the issue file cannot be written:

1. Say what failed
2. Still complete investigation
3. Output the full drafted issue in the chat
4. Do not pretend the file was written
