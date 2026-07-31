# Solve-Ready Linear Issue Template (project-review)

Use this structure for **every** leaf produced by `/project-review`.  
Copy into Linear `description` with **literal markdown newlines**. Omit a section only if truly N/A (note why).

This matches the `/issue` quality bar and adds review metadata so `/solve` can implement with light drift verification only.

```markdown
## Summary

<2–4 sentences: problem + impact + where in the product>

## Current behavior

- <what exists or happens today>
- Evidence: route `<path>`, UI text “…”, repro steps if applicable
- Code evidence: `<path>` (`SymbolName`, ~Lstart–end) <one-line note>

## Expected behavior

- <testable outcome 1>
- <testable outcome 2>

## Suspected root cause / scope

<hypothesis grounded in code pin; mark uncertainty clearly. For pure visual/content, state “presentation only” and still pin the owning component.>

## Code map

| Path | Role | Notes |
|------|------|-------|
| `apps/web/...` | Entry / UI | `ComponentName` ~L.. |
| `apps/web/...` | API / action | `handler` ~L.. |
| `packages/...` | Schema / util | … |

Primary package/app: `<apps/web | apps/native | packages/…>`

## Implementation notes

- Recommended approach: <bullets>
- Reuse: <existing pattern/file/surface to mirror>
- Design tokens / values if known: <or “match <surface>”>
- Data/migrations: <none | describe>

## Acceptance criteria

- [ ] <observable criterion>
- [ ] <edge case if relevant>
- [ ] <error/empty state if relevant>
- [ ] No regression in <related flow>

## Verification

Commands (from repo root / `AGENTS.md` when applicable):

- `<typecheck or build command for touched package>`
- Manual: <steps to confirm fix in UI or API>

## Drift check (before implementing)

Re-verify these anchors; if they still match, skip full re-investigation:

- [ ] `<path>` still exists and owns <behavior>
- [ ] `<export or route>` still named `<name>`
- [ ] `<related test or script>` still the right verification entrypoint

Snapshot context: reviewed at <ISO date>, mode `<fast|deep>`.

## Out of scope / do not change

- <explicit boundaries>
- <unrelated surfaces or refactors to avoid>

## Risks / blockers

- <none | credentials | related tickets | provider limits>

## Platform / stack

- Canonical targets: <e.g. Neon Postgres, or none specific>
- Must not use / abandoned for this work: <or none>
- Migration dependency: <TEAM-n if this waits on a cutover, or none>

## Related

- Parent epic: <title / id when known>
- Related: <TEAM-n or none>
- blockedBy: <TEAM-n or none>
- Duplicate of: <none — duplicates should not be filed>

## Supersedes (only when this ticket replaces earlier work)

If none: `- none`

### Full supersede

- **Mode:** full
- **Issues:** TEAM-100
- **Override scope:** …
- **Do not preserve:** …

### Partial supersede

- **Mode:** partial
- **Issues:** TEAM-100
- **Override scope:** paths / behaviors …
- **Preserve:** …
- **Do not preserve:** …

## Review metadata

- Lens: <Completeness | Functional | Edge | UI | Hierarchy | Taste | Responsiveness | A11y | Performance | Content | Cross-feature>
- Priority: <P0 | P1 | P2>
- Surface: <route / page / flow>
- Class: <foundation | feature | polish | content | a11y>
- Evidence: <screenshot / URL / console note / code-only>

## Assumptions

- <inferences made because docs or live access were incomplete>
```

---

## Title conventions

- Bug: `Fix [specific broken behavior] on [surface]`
- Missing feature: `Add [exact capability] to [surface]`
- UI consistency: `Align [component/surface] with design system / existing [pattern]`
- Taste / motion: `Add [exact transition] to [component]` or `Improve hierarchy on [page] so [primary element] dominates`
- Empty state: `Handle empty state for [list or view]`
- Content: `Rewrite [element] copy on [surface]`
- A11y: `Fix [specific a11y failure] on [surface]`

Titles: short, specific, searchable. Include the surface name. No trailing period. No vague titles like “Polish dashboard” or “Fix bug”.

---

## Epic body (parent only)

Epics are packaging, not implementable work. Keep short:

```markdown
## Review pass

- Mode: <fast|deep>
- Surface: <…>
- Date: <YYYY-MM>
- Intent: Package solve-ready leaves from agentic project review.
- **Do not implement this epic shell.** `/solve` expands to eligible children.

## Leaves

(filled after create, or leave empty — children are the work)

## Notes

- Filed unassigned in Backlog/Todo for `/solve`.
```
