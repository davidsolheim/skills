# Walk protocol

How to inventory, drive, and debug front-facing UI. Findings go to local
candidate files. Linear happens only after the coverage gate.

---

## Inventory (front-facing only)

Build `coverage.md` before opening the browser.

Sources (parallelize):

1. Route/page files: `src/routes/**`, `app/**/page.tsx`, `pages/**`, native
   navigators if this repo has a user-facing client.
2. Primary nav, footer, tabs, command palette, sitemap, marketing CTAs.
3. Auth gates and feature flags that hide screens.

**Include:** marketing, login, app chrome, settings, empty/error pages the user
can open, onboarding, legal pages linked from the product, public share/preview
views.

**Exclude (`not_front`):** `/api/*`, `createServerFn` modules, webhooks, cron,
CLI-only, install-PWA chrome the user did not ask to walk, Storybook unless it
is the product.

Each row:

```text
path: /studio
file: src/routes/studio.tsx
auth: public | signed-in | either
nav: header | in-app | linked-only | orphan
status: pending
```

Orphans (routable but not in nav) stay in the walk — they are often bugs or ideas.

---

## Live URL

Prefer, in order: `--url` → documented preview → local app already serving →
start via repo scripts and wait for health.

Do not invent a public production URL when local/preview exists. If you must
use production: no real payments, no email blasts, no destructive admin.

---

## Browser driver

Use the first tool that can **navigate, click, type, and read console**:

1. chrome-devtools MCP
2. agent-browser (`snapshot -i`, click/fill, `console`, `errors`)
3. browser-use
4. Playwright (a repo smoke script is load-only if it only screenshots —
   extend with a real script or MCP; a single screenshot is not enough)

Reconnect/reload once on a first-load timeout or stale-deps 504. Then continue.

Viewport defaults: **1280×800** (desktop) and **375×812** (mobile). Primary
journeys get both unless `--desktop-only` / `--mobile-only`.

---

## Per-surface loop (required)

For each `pending` unit:

1. **Go** to the URL. Wait for load (network idle or visible main content).
2. **Broken load?** Blank body, error boundary, 404 that nav promised, infinite
   spinner after a reasonable wait → `broken_load` + bug finding. Still note
   console/network.
3. **See.** Title, primary heading, obvious layout clip, overlapping chrome
   (including platform banners).
4. **Debug.** Console errors/warnings that fire on load. Failed XHR/fetch
   (4xx/5xx) for this surface.
5. **Interact.** Snapshot interactive elements. Then:
   - Click every **primary** CTA and in-page tab/segment on this screen
   - Open dialogs/drawers and close them
   - Forms: submit empty; one invalid value; one valid path **if it is safe**
     (no paid checkout, no live email to strangers)
   - Lists: search/filter if present; click one row if it goes somewhere
   - Follow in-product links that leave this URL (add those units if missing)
6. **States.** Empty, loading, error, unauthorized — trigger when cheap
   (filter-to-zero, signed-out hit on a gated URL). Else infer from code and
   still file if the UI has no branch.
7. **Viewport.** On primary surfaces, repeat a 30-second pass at the other
   viewport (nav overflow, tap targets, horizontal scroll).
8. Mark `walked` (or `blocked_auth` / `blocked_flag` with the reason).
9. Write findings **now**, not at the end of the whole site.

A surface with no findings is still `walked`. Do not invent fake tickets.

### Auth

- Public first.
- If login UI exists and this environment already has a session or a documented
  test sign-in that does **not** require the user to paste a password, walk
  signed-in chrome.
- Never put credentials in Linear. Never ask the user to “open localhost and
  log in for me” as a substitute for walking public pages.
- OAuth popups: complete them if the tool can; otherwise `blocked_auth` those
  post-login routes.

### Safe interaction

Do **not**: place real orders, delete other people’s data, send campaign email,
toggle production feature flags, or spam third-party APIs.

Do: client validation, local draft save, filters, pagination, copy buttons,
sign-out, harmless settings you can revert.

---

## What to look for (search pattern, not a quota)

| Lens | Examples |
|------|----------|
| Functional | Dead button, wrong page, save no-ops, lost form state, 500 on submit |
| Debug | Uncaught exception, failed `/api/*` the UI ignores, hydration mismatch |
| Edge | Empty list with no empty state; error with no retry; invalid never explained |
| Nav | Footer link 404; two labels for one destination; unreachable signed-in home |
| UI / layout | Clip, overlap, unreadable contrast, inconsistent control chrome vs sibling |
| Content | Lorem, placeholder “TODO”, truncated legal, misleading CTA copy |
| A11y | Unlabeled icon button, unreachable dialog, missing name on input |
| Responsive | Desktop-only nav with no mobile path; 375px horizontal scroll on core |
| Idea | Capability this surface implies but does not offer (export, undo, share) |

Do not file a ticket per lens per page. File **observable** problems and
**concrete** ideas. See [`finding-kinds.md`](finding-kinds.md).

---

## Evidence per finding

Minimum:

- Route / URL
- Viewport
- What you did (click/type/submit)
- What happened (copy, status, console line)
- Owning file if already obvious

Optional screenshot path: `screenshots/walk-$RUN_ID/<slug>.png` (workspace,
never `/tmp` for evidence you will cite).

Do not paste full HAR or 200-line console dumps into Linear. One error line +
request path is enough.

---

## Coverage gate

The walk is not done while any unit is `pending`.

Allowed terminals: `walked` | `blocked_auth` | `blocked_flag` | `broken_load` | `not_front`.

`broken_load` still counts as covered **and** must produce a bug leaf (or a
duplicate skip against the board snapshot).

If a worker slice fails: re-queue those units; do not mark them walked.

---

## Parallel slices (optional)

When front-facing units **> 15**:

- Orchestrator writes coverage + live URL + auth notes into scratch
- Spawn ≤ 4 `general-purpose` workers, disjoint route lists, isolation `none`
- Each worker follows this protocol and writes `issue-candidates/_inbox/`
- Workers **never** call Linear and **never** edit app source
- Orchestrator re-walks any unit the worker left `pending` or `failed`

---

## Anti-patterns

- `curl` 200 → `walked`
- Home + login only, then file a “whole app” ticket
- Screenshot mosaic with no clicks
- Walking Storybook instead of the product
- “I’ll file from the route tree without the browser because HMR is slow”
