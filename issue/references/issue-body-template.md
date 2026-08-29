# Linear issue body template

Copy this structure into `linear__save_issue` `description`. Use literal markdown newlines. Omit a section only if truly N/A (note why).

```markdown
## Intensity

Stamp required. Canonical: [`../../docs/intensity.md`](../../docs/intensity.md).
`/solve` and `/prb` read `Band:` as the effort/panel default.

- Band: `<light | standard | heavy | critical>`
- Why: `<one line: risk class>`
- Proof: `<on | n/a>`

## Summary

<2–4 sentences: problem + impact + where in the product>

## User report

> <original user description, lightly cleaned>

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
| `apps/web/...` | Entry / UI | `ComponentName` ~L.. |
| `apps/web/...` | API / action | `handler` ~L.. |
| `packages/db/...` | Schema / query | `tableName` |

Primary package/app: `<apps/web | apps/native | packages/…>`

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

Commands (from repo root / `AGENTS.md` when applicable):

- `<typecheck or build command for touched package>`
- Manual: <steps to reproduce fix in UI or API>

### Runtime proof

Fill when the leaf is user-visible, auth, billing, public API, schema, or a
shared helper (`../../docs/prove-it-works.md`). Omit with `- N/A: docs/comments only`.

- Surface to drive: `<route / CLI / METHOD path>`
- Visual reference (UI): `<sibling route or screenshot or n/a>`
- Blast-radius fact (shared/auth/schema): `<one fact + how to run it, or n/a>`
- Observed end state that proves done: `<what the driver must see>`

## Drift check (before implementing)

Re-verify these anchors; if they still match, skip full re-investigation:

- [ ] `<path>` still exists and owns <behavior>
- [ ] `<export or route>` still named `<name>`
- [ ] `<related test or script>` still the right verification entrypoint

Snapshot context: branch `<name if known>`, investigated at <ISO date>.

## Risks / blockers

- <none | credentials | related tickets | provider limits>

## Platform / stack

- Canonical targets: <e.g. Neon Postgres, Vercel Edge ingest>
- Must not use / abandoned for this work: <e.g. ClickHouse, Convex, or none>
- Migration dependency: <TEAM-n if this feature waits on a cutover, or none>

## Related

- Related: <TEAM-n or none>
- Duplicate of: <none>

## Supersedes (required when this ticket replaces earlier work)

Use this when later product decisions replace or narrow earlier tickets. `/solve` and implementers treat this as authority for selective override.

### Full supersede (skip older open ticket; replace its surface)

```markdown
## Supersedes
- **Mode:** full
- **Issues:** TEAM-100, TEAM-101
- **Override scope:** entire prior surface for <feature>
- **Do not preserve:** <old AC / pattern to abandon>
```

### Partial supersede (keep older ticket for non-overridden parts)

```markdown
## Supersedes
- **Mode:** partial
- **Issues:** TEAM-100
- **Override scope:**
  - paths: `apps/web/...`, `packages/...`
  - behaviors: <what changes>
- **Preserve:** <what from TEAM-100 must still hold>
- **Do not preserve:** <old AC to abandon inside override scope>
```

If this ticket does **not** replace earlier work: omit the section or write `- none`.

## Assumptions

- <inferences made from a brief report>
```
