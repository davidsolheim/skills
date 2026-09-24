# Linear issue body template

Canonical body for `/issue`, `/issues`, `/start` V1 leaves, `/project-review`, and `/walk` leaves. Do not
fork this file. Skill-specific extras:

- `/issues` — fill **Batch metadata**; quote the dump bullet in User report
- `/start` — fill **Batch metadata** (`batch: start-v1-<slug>`); quote the VISION.md V1 bullet in User report; investigate the new starter checkout
- `/project-review` — append Review metadata from
  [`../../project-review/references/issue-template.md`](../../project-review/references/issue-template.md)
- `/walk` — append Walk metadata from
  [`../../walk/references/walk-metadata.md`](../../walk/references/walk-metadata.md); User report = live UI observation
- `/issue` — omit Batch metadata, Review metadata, and Walk metadata

Copy into `linear__save_issue` `description`. Use **literal markdown newlines**
(not `\n` escape sequences). Omit a section only if truly N/A — and write why
(e.g. `- N/A: no data model change`).

Every leaf must be **self-contained** (no “see L1 / see epic”). See
[execution-ready-bar.md](execution-ready-bar.md).

```markdown
## Implementer contract

- You are implementing **this ticket only** (not a parent epic, not siblings). Do not expand scope.
- If `blockedBy` is set and those issues are not Done, do not start this leaf.
- Prefer the **step-by-step plan** and **file-by-file changes** below over inventing a new design.
- Before coding: run the **Drift check**. If anchors still match, do **not** re-research the whole area — implement.
- If drift broke the plan (paths/symbols gone), stop and report; do not freestyle a rewrite.
- Smallest complete change that meets **Acceptance criteria**. Mirror existing patterns; do not introduce new libraries or architectural layers unless this ticket says so.
- Never commit secrets, `.env` values, or real credentials.

## Occupancy (WCP)

Shared local `dev` may have other writers. Spec: [watercoolerprotocol.com](https://watercoolerprotocol.com). Skill: `water-cooler-protocol`.

- Primary write path: `<path>` (the file this leaf occupies first)
- Primary symbol: `<Symbol>`
- Other write paths: `<path>`, … (or none)
- Barrels (lockfile / generated client / root schema): `<path>` or none
- Sibling overlap: disjoint | shares `<path>` with L# / TEAM-n (occupancy serializes; mint `blockedBy` only if AC depends)

Implementer: name yourself with `wcp name <id>` and export `WCP_AGENT` and `WCP_NAME_TOKEN`. Write the test first (`// WCP <id>: <path> …`) and do not claim it. Write a new file with no claim. For a file that already existed: look → acquire --test → write-ok → re-read disk → edit → release. Never rewind sibling hunks. Never push `origin/dev`.

Docs-only / no source edit: `- N/A: no application writes`.

## Intensity

Stamp required. Canonical: [`../../docs/intensity.md`](../../docs/intensity.md).
`/solve` and `/prb` read `Band:` as the effort/panel default. Do not omit.
Do not invent a fifth band. Classify on risk class, not ticket length.

- Band: `<light | standard | heavy | critical>`
- Why: `<one line: risk class, e.g. isolated UI; one-file component>`
- Proof: `<on | n/a>`

## Summary

<2–4 sentences: problem + user/business impact + where in the product (route/screen/API). Write so someone who never used the app understands the slice.>

## User report

> <original user description, dump bullet, or review finding — lightly cleaned>

## Current behavior

- <what happens today, step by step if a flow>
- Evidence: `<path>` (`SymbolName`, ~Lstart–end) — <one-line note>
- Evidence: `<path>` (`…`) — <…>
- <error text / wrong UI / wrong data if known>

## Expected behavior

- <concrete, testable outcome 1 — UI copy, API status, data shape, etc.>
- <outcome 2>
- <non-goals restated briefly if easy to overbuild>

## Suspected root cause / scope

<hypothesis grounded in code reads. Mark uncertainty: "likely" vs "confirmed".>
Scope boundary: <what is in vs adjacent systems left alone>.

## Code map

| Path | Role | Symbols / notes |
|------|------|-----------------|
| `apps/web/...` | Entry / route / page | `PageName` ~L.. |
| `apps/web/...` | UI component | `ComponentName` props: `…` ~L.. |
| `apps/web/...` | Action / API / handler | `handlerName` ~L.. |
| `packages/db/...` | Schema / query | `tableName`, columns: `…` |
| `…` | Test to extend or mirror | `describe(…)` |

Primary package/app: `<apps/web | apps/native | packages/…>`
Owning monorepo path (if monorepo): `<…>`

## Relevant contracts (types / APIs / data)

Paste only what the implementer must respect (trim aggressively):

- **Types / props / schema:**  
  `<TypeOrInterfaceName>` in `<path>` — fields: `…`
- **API / route:**  
  `METHOD /path` → request `{…}` → response `{…}` (or “see handler at …”)
- **DB / storage:**  
  table/collection `…`, relevant columns/fields `…`, ownership package `…`
- **Env / flags (names only):**  
  `PROCESS_ENV_NAME` — purpose; never paste values
- **Auth / tenancy constraints:**  
  <e.g. must filter by orgId; server-only; etc.>

## Code anchors (excerpts)

Short excerpts of **non-obvious** logic the change hinges on. Prefer 5–40 lines
total per excerpt. Label path + symbol. Omit if the change is trivial and the
code map is enough.

### `<path>` — `symbol` (~Lstart–end)

```ts
// …trimmed excerpt only…
```

### Pattern to mirror

- **Mirror:** `<path>` (`SymbolName`) — copy <structure / error handling / loading state / query pattern>
- **Why:** <one line>

## Step-by-step implementation plan

Ordered. Cheap models follow this literally unless drift blocks a step.

1. <prep: types/schema/migration if any>
2. <edit primary handler/component — what to change in plain language>
3. <wire UI or caller>
4. <tests>
5. <docs only if this ticket requires it>
6. <verify with commands in Verification>

## File-by-file changes

| Path | Action | What to change |
|------|--------|----------------|
| `…` | edit | <specific: add field X, fix condition Y, call Z> |
| `…` | create | <new file purpose; export name; who imports it> |
| `…` | edit test | <cases to add> |
| `…` | do not touch | <listed so scope stays tight — or use Do not touch section> |

## Do not touch / out of scope

- <paths, packages, or refactors to avoid>
- <product behaviors explicitly not changing>
- <no drive-by renames, dependency upgrades, or formatting-only sweeps>

## Acceptance criteria

Every box must be pass/fail by an outsider with no extra product context:

- [ ] <observable criterion — user-visible or API-visible>
- [ ] <edge case: empty / null / unauthorized / mobile if relevant>
- [ ] <error or loading state if relevant>
- [ ] <data integrity / tenancy if relevant>
- [ ] No regression in <related flow named concretely>
- [ ] Verification commands in this ticket pass

## Test plan

**Automated (prefer):**

- File: `<path to test file to add or extend>`
- Cases:
  - <input → expected>
  - <edge → expected>

**Manual:**

1. <preconditions: user role, seed data, feature flag>
2. <steps>
3. <expected result>
4. <regression spot-check>

If no automated test is practical: say why and make manual steps airtight.

## Verification

Copy-paste commands from this repo (`AGENTS.md`, package scripts). Adjust paths
to the owning package:

- ` <typecheck command> `
- ` <unit/integration test command if applicable> `
- ` <build command if AGENTS requires it for this kind of change> `
- Manual: <one-line summary pointing at Test plan>

### Runtime proof (`/solve` / `/prb` / `/yeet` drive this — not a new slash)

Fill when the leaf is user-visible, auth, billing, public API, schema, or a
shared helper. Omit with `- N/A: docs/comments only` otherwise.
Canonical: [`../../docs/prove-it-works.md`](../../docs/prove-it-works.md).

- Surface to drive: `<route / CLI / METHOD path>`
- Project verify skill / feature map: `<path or none — do not invent a skill>`
- Visual reference (UI): `<sibling route or screenshot or n/a>`
- Blast-radius fact (shared/auth/schema): `<one fact + how to run it, or n/a>`
- Observed end state that proves done: `<what the driver must see>`

## Drift check (before implementing)

Re-verify these anchors; if they still match, **skip full re-investigation** and implement:

- [ ] `<path>` still exists and owns <behavior>
- [ ] `<export | route | symbol>` still named `<name>`
- [ ] `<type | schema | column>` still shaped as described under Contracts
- [ ] `<mirror path>` still is the right pattern to copy
- [ ] `<test or script>` still is the right verification entrypoint

Snapshot: investigated at <ISO date>, branch `<if known>`, HEAD hint `<short sha optional>`.

## Risks / blockers

- <none | migration risk | credentials needed | rate limits | related open tickets>
- Rollback note: <optional one-liner>

## Platform / stack

- Canonical targets: <e.g. Neon Postgres, Next.js App Router, Vercel, or none specific>
- Must not use / abandoned for this work: <e.g. ClickHouse, Convex, or none>
- Migration dependency: <TEAM-n if this waits on a cutover, or none>

## Related

- Parent epic: <title / id when known, or none>
- Related: <TEAM-n or none>
- blockedBy: <TEAM-n or none>
- Duplicate of: <none — duplicates should not be filed>

## Supersedes (required when this ticket replaces earlier work)

Use when later product decisions replace or narrow earlier tickets. Intake
must also **retire unstarted** full-contradiction issues (Canceled or
Duplicate) — see [direction-conflict.md](direction-conflict.md). `/solve`
treats this section as override authority; it does **not** replace retiring X.

### Full supersede

- **Mode:** full
- **Issues:** TEAM-100, TEAM-101
- **Override scope:** entire prior surface for <feature>
- **Do not preserve:** <old AC / pattern to abandon>
- **Board action:** Canceled TEAM-100, Duplicate TEAM-101 of this issue

### Partial supersede

- **Mode:** partial
- **Issues:** TEAM-100
- **Override scope:**
  - paths: `…`
  - behaviors: <what changes>
- **Preserve:** <what from TEAM-100 must still hold>
- **Do not preserve:** <old AC to abandon inside override scope>
- **Board action:** left open (residual) | Canceled (residual empty)

If this ticket does **not** replace earlier work: `- none`

## Batch metadata (`/issues` only — omit otherwise)

- temp_id: L#
- class: <foundation | feature | polish | content | a11y | chore>
- batch: <short name for this `/issues` run>

## Assumptions / pre-decided

Decisions the intake agent already made so the implementer does **not** ask or guess:

- <inference from a brief report>
- <UX choice when the user was vague>
- <API shape choice consistent with existing code>

If a decision truly needs a human and blocks coding, put it under Risks as a
blocker instead of filing a pretend-ready ticket.
```

### Section priority when time-boxed

Never skip: Intensity, Occupancy (WCP), Summary, Current/Expected, Code map, Step-by-step plan,
File-by-file, Acceptance criteria, Verification, Drift check, Assumptions.

May shorten: Code anchors (if truly trivial), Test plan automated cases
(but then manual must be strong), Supersedes (write `- none`).

`/issues`: Batch metadata required. Epic parent is packaging only.
