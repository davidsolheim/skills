# Project-review leaf body

Canonical solve-ready body:
[`../../issue/references/issue-body-template.md`](../../issue/references/issue-body-template.md)

Use that structure for **every** leaf. Quality bar:
[`../../issue/references/execution-ready-bar.md`](../../issue/references/execution-ready-bar.md).

Then append the sections below. User report = the discovery finding (not a
human dump). Omit `/issues` Batch metadata. Direction conflicts:
[`../../issue/references/direction-conflict.md`](../../issue/references/direction-conflict.md).

Copy into Linear `description` with **literal markdown newlines**.

```markdown
## Review metadata

- Lens: <Completeness | Functional | Edge | UI | Hierarchy | Taste | Responsiveness | A11y | Performance | Content | Cross-feature>
- Priority: <P0 | P1 | P2>
- Surface: <route / page / flow>
- Class: <foundation | feature | polish | content | a11y>
- Evidence: <screenshot / URL / console note / code-only>
```

## Title conventions

- Bug: `Fix [specific broken behavior] on [surface]`
- Missing feature: `Add [exact capability] to [surface]`
- UI consistency: `Align [component/surface] with design system / existing [pattern]`
- Taste / motion: `Add [exact transition] to [component]` or `Improve hierarchy on [page] so [primary element] dominates`
- Empty state: `Handle empty state for [list or view]`
- Content: `Rewrite [element] copy on [surface]`
- A11y: `Fix [specific a11y failure] on [surface]`

Titles: short, specific, searchable. Include the surface name. No trailing
period. No vague titles like “Polish dashboard” or “Fix bug”.

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
