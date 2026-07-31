# Expanded Review Checklist

Use during Phase 2 discovery (fast) and inside **deep review workers** for each assigned inventory unit. Not every item produces an issue — only high-signal findings that will pass quality gates.

**Tags:** `[F]` = required or strongly expected in **fast** mode · `[D]` = deep mode (or when already on a primary surface in fast)

**Deep workers:** For every assigned unit, run the full `[F]` + `[D]` set that applies to that unit’s kind (route vs API vs component). Recording “reviewed, 0 findings” after a real pass is correct — do not invent tickets to satisfy the checklist.

**Coverage ≠ tickets:** Exhaustive deep review means every unit is inspected; it does not mean one Linear issue per unit or per checkbox.

---

## Completeness

- [ ] `[F]` Core user journeys from intended state / docs / prior tickets exist end-to-end
- [ ] `[D]` Secondary intended features exist (filters, sorting, bulk actions, exports, settings subpages)
- [ ] `[D]` Admin or internal tools present when in scope
- [ ] `[F]` Mobile / responsive versions of key flows exist when product requires them
- [ ] `[D]` Feature flags / env toggles do not leave half-dead UI in default config

## Functional correctness

- [ ] `[F]` Happy-path flows complete without errors
- [ ] `[F]` Form validation behaves correctly (client + server where both exist)
- [ ] `[F]` Data displayed matches source of truth for primary views
- [ ] `[F]` Mutations (create / update / delete) persist and reflect as designed
- [ ] `[F]` Auth-gated routes and actions respect permissions on primary paths
- [ ] `[F]` No console errors or unhandled rejections on primary paths (when live walk available)
- [ ] `[D]` Secondary flows and edge role combinations work
- [ ] `[D]` Idempotent / double-submit behavior is safe on critical mutations

## Edge, empty, and error states

- [ ] `[F]` Zero-data / first-time empty states on core lists/views are intentional and helpful
- [ ] `[F]` Loading states exist for primary async sections (skeletons or spinners as appropriate)
- [ ] `[D]` Network / server error states are clear and offer recovery
- [ ] `[D]` Invalid input and permission-denied states are handled
- [ ] `[D]` Extremely long content or many items do not break layout
- [ ] `[D]` Partial failure (one widget fails) does not blank the whole page when avoidable

## UI consistency & design system

- [ ] `[F]` High-visibility pages use consistent spacing scale
- [ ] `[F]` Primary buttons / inputs / modals use shared variants (not one-offs on main CTAs)
- [ ] `[D]` Typography hierarchy consistent (headings, body, captions, labels)
- [ ] `[D]` Color usage matches brand / semantic tokens (primary, muted, destructive, success)
- [ ] `[D]` Border radius, shadows, borders consistent across similar components
- [ ] `[D]` Icons from the same set and sized consistently
- [ ] `[D]` Design-token drift: hard-coded colors/spacing where tokens exist

## Visual hierarchy & information density

- [ ] `[F]` Most important element on each primary page is visually dominant
- [ ] `[F]` Primary actions are easy to find; destructive actions restrained
- [ ] `[D]` Secondary information correctly demoted
- [ ] `[D]` Related items grouped; unrelated items separated
- [ ] `[D]` Density matches product tone (not cramped or sparse vs sibling pages)

## Interaction, motion, and feel

- [ ] `[F]` Hover / focus / active states exist on primary interactive elements
- [ ] `[D]` Transitions on modals, drawers, menus, expanding sections present and consistent
- [ ] `[D]` Loading feedback appears quickly enough to feel responsive
- [ ] `[D]` No janky animations or layout thrashing
- [ ] `[D]` Micro-interactions support intended emotional tone

## Responsiveness

- [ ] `[F]` Core layouts work at mobile and desktop
- [ ] `[F]` Primary actions usable on small screens
- [ ] `[D]` Tablet breakpoint
- [ ] `[D]` Touch targets adequately sized
- [ ] `[D]` Horizontal overflow avoided
- [ ] `[D]` Safe areas / mobile nav patterns

## Accessibility

- [ ] `[F]` Clear failures only in fast: unlabeled critical controls, invisible focus on primary flows, broken keyboard on main CTA path
- [ ] `[D]` Text contrast meets minimum standards
- [ ] `[D]` Focus visible and logical order
- [ ] `[D]` Interactive elements keyboard reachable
- [ ] `[D]` Images/icons have appropriate alt / labels
- [ ] `[D]` Form fields have associated labels
- [ ] `[D]` Dynamic updates announced when necessary

## Performance & perceived performance

- [ ] `[F]` Obvious jank, multi-second blank states, or major CLS on primary loads
- [ ] `[D]` Initial load of key pages acceptable
- [ ] `[D]` Skeletons or progressive loading where helpful
- [ ] `[D]` Heavy tables/charts/lists remain usable

## Content & microcopy

- [ ] `[F]` No placeholder or lorem text on production paths
- [ ] `[F]` Broken/missing labels on primary CTAs and empty states
- [ ] `[D]` Headings clear and consistent
- [ ] `[D]` Empty/error messages helpful rather than technical
- [ ] `[D]` Button text accurately describes the action
- [ ] `[D]` Tone matches product voice

## Cross-feature consistency

- [ ] `[F]` Blatant only: same concept, wildly different language or UI on two primary surfaces
- [ ] `[D]` Shared components behave the same way in different places
- [ ] `[D]` Same concepts use the same language and visual treatment app-wide

## Auth & session (extra)

- [ ] `[F]` Sign-in / sign-out / session expiry on critical paths
- [ ] `[D]` Role-based UI does not expose forbidden actions as if enabled
- [ ] `[D]` Permission-denied empty states are intentional

## Navigation & stubs (extra)

- [ ] `[F]` Nav links on primary chrome resolve to real routes
- [ ] `[D]` Dead ends and “coming soon” half-wires are intentional or ticketed
- [ ] `[D]` Production paths free of debug-only UI

## “Intended but stubbed” signals (code)

- [ ] `[D]` `TODO` / `FIXME` / `HACK` on user-facing paths
- [ ] `[D]` `notImplemented`, `throw new Error("TODO")`, empty handlers
- [ ] `[D]` Placeholder components (`ComingSoon`, `lorem`, hard-coded fake data in prod builds)
- [ ] `[F]` Routes registered in nav but missing page implementation

---

## Signal filter (before promoting to a ticket)

**Promote** if any of:

- Breaks or blocks a primary journey
- Missing intended feature with clear user value
- High-visibility inconsistency or hierarchy failure
- Clear a11y failure on a primary control
- Placeholder/lorem/stub in production path

**Drop** if:

- Pure nit (1px misalignment no one will notice)
- Speculative rewrite without evidence of user impact
- Duplicate of another candidate or open Linear issue
- Cannot be made concrete after one taste-conversion attempt (and not escalated)

Fast mode: when in doubt, **drop** or leave as handoff note.  
Deep mode: when in doubt, require code pin + concrete AC or drop (during **local** cleanup on disk — not via Linear thrash).
