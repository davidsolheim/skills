# Linear issue body template (`/issues` leaves)

Same quality bar as `/issue`. Copy into `linear__save_issue` `description`
with **literal markdown newlines**. Omit a section only if truly N/A (note why).

```markdown
## Summary

<2–4 sentences: problem + impact + where in the product>

## User report

> <the specific bullet / fragment from the multi-item dump>

## Current behavior

- <what happens today>
- Evidence: `<path>` (`SymbolName`, ~Lstart–end) <one-line note>

## Expected behavior

- <testable outcome 1>
- <testable outcome 2>

## Suspected root cause / scope

<hypothesis grounded in code reads; mark uncertainty clearly>

## Code map

| Path | Role | Notes |
|------|------|-------|
| `…` | Entry / UI | `ComponentName` ~L.. |
| `…` | API / action | `handler` ~L.. |
| `…` | Schema / query | `tableName` |

Primary package/app: `<apps/web | packages/api | packages/db | …>`

## Implementation notes

- Recommended approach: <bullets>
- Reuse: <existing pattern/file to mirror>
- Avoid / out of scope: <bullets>
- Data/migrations: <none | describe>

## Acceptance criteria

- [ ] <observable criterion>
- [ ] <edge case>
- [ ] <error/empty state if relevant>
- [ ] No regression in <related flow>

## Verification

Commands (from package `AGENTS.md` / README when applicable):

- `<typecheck or build for touched package>`
- Manual: <steps>

## Drift check (before implementing)

Re-verify these anchors; if they still match, skip full re-investigation:

- [ ] `<path>` still exists and owns <behavior>
- [ ] `<export or route>` still named `<name>`
- [ ] `<related test or script>` still the right verification entrypoint

Snapshot context: investigated at <ISO date>, batch `<short plan title>`.

## Risks / blockers

- <none | credentials | related tickets | provider limits>

## Platform / stack

- Canonical targets: <e.g. Neon Postgres, Vercel, or none specific>
- Must not use / abandoned for this work: <or none>
- Migration dependency: <TEAM-n if waits on cutover, or none>

## Related

- Parent epic: <title / id when known, or none>
- Related: <TEAM-n / temp siblings, or none>
- blockedBy: <TEAM-n / temp siblings, or none>
- Duplicate of: <none — duplicates should not be filed>

## Supersedes (when this ticket replaces earlier work)

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

## Batch metadata

- temp_id: L#
- class: <foundation | feature | polish | content | a11y | chore>
- batch: <short name for this /issues run>

## Assumptions

- <inferences made from a brief bullet>
```
