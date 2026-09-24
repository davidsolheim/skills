---
name: vercel-flags
description: >-
  Wire and operate Vercel Flags via the Flags SDK (@flags-sdk/vercel),
  Flags Explorer discovery, identify/targeting, and the vercel flags CLI.
  Use when the user says /vercel-flags, "add feature flags", "Vercel Flags",
  "feature flag this", "Flags Explorer", or "roll out with flags". Prefer
  this over ad-hoc env booleans for product rollouts and targeting.
argument-hint: setup | add | wire | toggle | audit
---

# Vercel Flags

Operate feature flags as code + dashboard/CLI config. Default to the Flags
SDK with `@flags-sdk/vercel`, Flags Explorer, and `vercel flags`.

## 1. Default stack

| Piece | Choice |
| --- | --- |
| App framework | Next.js App Router (`flags/next`) |
| Adapter | `@flags-sdk/vercel` + `vercelAdapter()` |
| Project link | `vercel link` then `vercel env pull` |
| Explorer | Discovery route at `app/.well-known/vercel/flags/route.ts` |
| Ops CLI | `vercel flags` (create, enable/disable, rules, rollout) |

## 2. Docs

- Quickstart: https://vercel.com/docs/flags/vercel-flags/quickstart
- Flags SDK: https://vercel.com/docs/flags/sdks/flags-sdk
- CLI: https://vercel.com/docs/cli/flags
- flags-sdk.dev: https://flags-sdk.dev
- Cheatsheet: [references/cli-cheatsheet.md](references/cli-cheatsheet.md)

## 3. Operating contract

- **Flags as code + config**: define flag declarations in repo; manage targeting,
  rules, and rollouts via dashboard or `vercel flags` CLI.
- **Never invent or commit secrets**: do not invent, print, or commit
  `FLAGS_SECRET`, SDK keys (`vf_server_*` / `vf_client_*`), or connection
  strings. Pull via `vercel env pull` / dashboard; keep them in env only.
- **Git branches**: long-lived remotes are `origin/main` and `origin/dev` ONLY.
  After checks pass, push `origin/dev` only with the WCP export below. Shipping to
  `main`/production needs explicit user approval via `/prb`. Confirm before
  production flag toggles unless the user already asked for them.
- **Occupancy:** before editing this checkout, follow [`../docs/wcp.md`](../docs/wcp.md)
  and skill `water-cooler-protocol`. Name yourself, lease pre-existing files for the
  edit burst, release before tests. Do not commit and do not stash while a source-file lease is live. `wcp look` before the session commit or push.
  `unset WCP_AGENT WCP_NAME_TOKEN` immediately before `git push`. Commit `.WCP/issues/`. Do not commit `.WCP/RUN.md`, `.WCP/run.sqlite`, or sqlite wal/shm.
- **Queue:** file follow-up work under `.WCP/issues/` ([`../docs/wcp-queue.md`](../docs/wcp-queue.md)). Do not call Linear.
- **Eval model**: prefer **server-side** evaluation (RSC / Route Handlers).
  Flags are **not** authorization — never use a flag alone as an authz gate.

## 4. Modes

| Mode | When | Outcome |
| --- | --- | --- |
| `setup` | Repo has no Flags SDK / Explorer | Packages, `flags.ts`, discovery route, env |
| `add` | New flag needed | Create via CLI + declare in code |
| `wire` | Flag exists; app must read it | Evaluate in RSC/handlers; ship on `dev` |
| `toggle` | Change targeting / on-off / rollout | CLI or dashboard; ask before production |
| `audit` | Health check | Inventory flags, secrets hygiene, dead code |

## 5. Phase 0 — Repo fit

Before changing code:

1. Confirm Next.js App Router (or note adapter differences).
2. Check for existing `flags` / `@flags-sdk/vercel` / `@vercel/flags` usage.
3. Confirm Vercel project link (`.vercel/project.json`) and env access.
4. Find current feature-gate patterns (env booleans, LaunchDarkly, etc.).
5. Prefer migrating ad-hoc env booleans to named Vercel Flags when the user
   wants rollouts, targeting, or Explorer.

## 6. Phase 1 — Setup

### Packages and project link

```bash
bun add flags @flags-sdk/vercel
vercel link
vercel env pull
```

Ensure `FLAGS_SECRET` (and any SDK key / connection string the project uses)
exist in Vercel env and land in `.env.local` via pull — never invent them.

### `flags.ts` with `vercelAdapter`

Example (adjust paths to the repo):

```ts
import { flag } from "flags/next";
import { vercelAdapter } from "@flags-sdk/vercel";

export const showNewCheckout = flag({
  key: "show-new-checkout",
  adapter: vercelAdapter(),
  // optional: defaultValue, identify, description
});
```

### Flags Explorer discovery route

Create `app/.well-known/vercel/flags/route.ts`:

```ts
import { createFlagsDiscoveryEndpoint, getProviderData } from "flags/next";
import * as flags from "@/flags";

export const GET = createFlagsDiscoveryEndpoint(async () => {
  return getProviderData(flags);
});
```

### Create a flag in the project

```bash
vercel flags create show-new-checkout --description "New checkout UI"
# or string/json kinds with --kind and --variant as needed
```

## 7. Phase 2 — Evaluate in RSC + identify / dedupe

Prefer server components / server code:

```tsx
import { showNewCheckout } from "@/flags";

export default async function CheckoutPage() {
  const enabled = await showNewCheckout();
  return enabled ? <NewCheckout /> : <LegacyCheckout />;
}
```

Identify entities for targeting (user/team) and dedupe when evaluating many
flags in one request:

```ts
import { dedupe } from "flags/next";
import { flag } from "flags/next";
import { vercelAdapter } from "@flags-sdk/vercel";

const identify = dedupe(async () => {
  // resolve from session/cookies/headers — never trust client alone
  const user = await getSessionUser();
  return {
    user: { id: user?.id, plan: user?.plan, email: user?.email },
  };
});

export const showNewCheckout = flag({
  key: "show-new-checkout",
  adapter: vercelAdapter(),
  identify,
});
```

## 8. Phase 3 — CLI control (ask before production)

Use `vercel flags` for enable/disable, rules, segments, split, and rollout.
See [references/cli-cheatsheet.md](references/cli-cheatsheet.md).

Examples:

```bash
vercel flags enable show-new-checkout --environment preview --message "QA"
vercel flags rules add show-new-checkout --environment production \
  --condition user.plan:eq:pro --variant on --message "Pro cohort"
vercel flags rollout show-new-checkout --environment production --by user.id \
  --stage 5,6h --stage 25,12h --stage 50,1d --message "Gradual rollout"
```

**Always ask before production toggles/rollouts** unless the user already
requested that specific production change.

## 9. Phase 4 — Wire checklist → push `dev`

1. Flag declared in code with stable `key` matching Vercel slug.
2. Evaluated server-side where the UX/API branch happens.
3. `identify` / targeting attributes wired if rules need them.
4. Discovery route present for Flags Explorer.
5. Env pulled locally; secrets not committed.
6. Preview/dev behavior verified (Explorer overrides when useful).
7. Tests or smoke path cover both on/off (or variants).
8. Run checks. If this session is the only writer, commit on the working branch only when `wcp look` shows no live source-file lease. Do not stash another writer's files.
9. **`unset WCP_AGENT WCP_NAME_TOKEN`**, then `git push origin dev` after checks (no approval needed for `dev`).
10. Do **not** merge/push `main` or change production flags without approval
    (`/prb` for main/prod ship; confirm for production toggles).

## 10. Phase 5 — Audit

- `vercel flags list` / `list --json` vs code declarations (orphans / missing).
- Discovery endpoint reachable; Explorer can list flags.
- No secrets in git history or committed `.env*`.
- Flags not used as authz; sensitive paths still enforce real permissions.
- Dead flags: archived or removed after code paths deleted.
- Production rules/rollouts have owners and revision messages.
- `vercel flags evaluations <flag>` for traffic sanity when needed.

## 11. Anti-patterns + when stuck

**Anti-patterns**

- Ad-hoc `process.env.FEATURE_X === "true"` for product rollouts when Vercel
  Flags is available.
- Client-only evaluation for security-sensitive branches.
- Treating a flag as authorization / entitlement enforcement.
- Inventing or committing `FLAGS_SECRET` / SDK keys.
- Pushing to `main` or toggling production without explicit approval.
- Creating long-lived remote branches other than `main` and `dev`.
- Mismatched flag `key` / CLI slug between code and dashboard.

**When stuck**

1. Re-read quickstart + CLI docs (links above).
2. Confirm `vercel link`, `vercel env pull`, and adapter package versions.
3. `vercel flags inspect <flag>` and Explorer discovery route.
4. Check identify payload matches rule entity attributes (`user.plan`, etc.).
5. Ask the user before any production change. File follow-up work under `.WCP/issues/`.

