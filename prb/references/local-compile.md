# /prb — Local compile + ship-set tests (pre-push)

Type errors and unit-test failures that the laptop can catch must **never** consume the `/prb` babysit window. Run this project’s **compile/build** and **ship-set tests** on local `dev` after Phase 1.5 is clean (or `--skip-review`) and **before** `git push origin dev`.

This is **not** runtime proof ([`../../docs/prove-it-works.md`](../../docs/prove-it-works.md)). A green matrix does not pass that bar. This is **not** the native/macOS ship product build ([`ship-product-build.md`](ship-product-build.md)). GitHub CI babysit still runs after push.

---

## 1. Discovery

From the git root, on local `dev` after Phase 1:

| Field | How to resolve |
|-------|----------------|
| `PKG` | Lockfile / `package.json#packageManager`: `bun.lock`/`bun.lockb` → `bun`; `pnpm-lock.yaml` → `pnpm`; `yarn.lock` → `yarn`; else `npm` |
| `LOCAL_COMPILE_CMD` | `package.json` `scripts.build` → `$PKG run build`. Else `scripts.typecheck` → `$PKG run typecheck`. Else the **build/typecheck** line in `AGENTS.md` / CI that does **not** need GitHub secrets. Prefer plain `build`, not `build:doppler`, unless AGENTS says `/prb` must wrap Doppler. |
| `LOCAL_TEST_CMD` | See §2 |
| `LOCAL_COMPILE_SKIP` | User passed `--skip-local-compile` → skip with loud warning |

**n/a (do not invent a toolchain):** docs/skill/markdown-only ship, or no build/typecheck/test scripts and no AGENTS verify commands.

**Env:** If compile fails on missing `AUTH_SECRET` / `DATABASE_URL` / similar, reuse **CI workflow stub env** or the project’s documented local stubs. Do **not** wrap Doppler for typecheck unless AGENTS requires it. Never print secret values.

---

## 2. Ship-set tests

`SHIP_PATHS` = `git -c core.quotepath=false diff --name-only origin/main...dev`

**Test files in the ship set:**

- Any path matching `*.test.*`, `*.spec.*`, or `__tests__/`
- For each changed `foo.ts` / `foo.tsx` / `foo.js` / `foo.jsx`, a sibling `foo.test.*` / `foo.spec.*` if it exists on disk

**Command:**

1. Project `scripts.test` is unit/integration (`bun test`, `vitest`, `jest`, `node --test`, …) and is what CI runs → **`$PKG run test`** (full suite). Subset-only misses load-order / `mock.module` isolation failures.
2. Else if ship-set test files exist and the runner accepts paths (`bun test a.test.ts b.test.ts`) → run those paths.
3. Else `scripts.test` is Playwright/Cypress/e2e-only → run ship-set unit files if any; do **not** start a full e2e battery unless AGENTS says `/prb` must.
4. No test script and no ship-set test files → tests `n/a` (compile still required when a build/typecheck command exists).

Example (Bun app whose CI is `bun run build` then `bun run test`):

```bash
bun run build
bun run test
```

---

## 3. When to run

| Step | When | Action |
|------|------|--------|
| Inventory | Phase 0 | Fill `LOCAL_COMPILE_CMD`, `LOCAL_TEST_CMD` |
| Pre-push | After Phase 1.5 **clean** (or `--skip-review`), **before** Phase 1C½ and Phase 1D | Run compile then tests |
| Failure | Non-zero exit | **Do not push**; fix on local `dev`; re-run this phase; do not open a PR; do not start babysit |
| Babysit fix | After re-review clean, **before** re-pushing `dev` | Re-run this phase on the updated tree |

`--skip-review` does **not** skip this phase.

### Skip

- `--skip-local-compile` only with explicit user intent (loud warning).
- Docs/markdown-only ship, or no discovered commands → `n/a`.

---

## 4. Execution

1. Working tree: on local `dev`; `origin/main` is an ancestor; no unresolved conflicts.
2. Run `LOCAL_COMPILE_CMD` from the git root (timeout: several minutes — `next build` is slow).
3. On success, run `LOCAL_TEST_CMD` when not `n/a`.
4. Either non-zero: capture a redacted log tail; **block push**.
5. Do not commit `.next/`, `dist/`, or coverage output.

Fix compile/test failures on local `dev` (commit if the ship needs the fix). Re-run this phase. Do **not** re-run the Phase 1.5 panel unless the product diff is large enough that babysit §8 would already re-review.

---

## 5. Report fields (Phase 5)

```markdown
**Local compile:** `$LOCAL_COMPILE_CMD` ok @ <time> | n/a | skipped (--skip-local-compile) | blocked (<reason>)
**Ship-set tests:** `$LOCAL_TEST_CMD` ok | n/a | skipped | blocked (<reason>)
```

---

## 6. Rationalizations

| Excuse | Reality |
|--------|---------|
| "CI will catch type errors" | That is the 15-minute window this gate exists to protect. |
| "/solve already typechecked" | Ship set may have grown. Run again on the **final** `dev` tree. |
| "Runtime proof is enough" | Proof drives the path. It does not typecheck `next build`. |
| "Ship-set tests means skip the suite" | When CI runs `bun test` / vitest / jest, run that full script. |
| "Build needs Doppler" | Plain `build` unless AGENTS says `/prb` must wrap Doppler. |
| "This is the macOS ship product build" | That is Phase 1C½. This phase is compile + tests. |

## Anti-patterns

- Pushing `dev` then watching CI fail on a type error `next build` would have shown locally
- Skipping this phase because the review panel was clean or `--skip-review` was set
- Wrapping `doppler run` around compile "to be safe" when CI uses stub env + plain `build`
- Treating this green matrix as runtime proof
- Running Playwright/Cypress as a substitute for unit tests, or skipping unit tests because e2e exists
- Inventing a compile toolchain in a docs-only repo
