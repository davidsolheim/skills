# Walk leaf extras

Canonical body:
[`../../issue/references/issue-body-template.md`](../../issue/references/issue-body-template.md)

Taste / vague ideas:
[`../../project-review/references/taste-to-concrete.md`](../../project-review/references/taste-to-concrete.md)

User report = the walk observation. Append after the canonical `/issue` sections.

```markdown
## Walk metadata

- Kind: <bug | idea | improvement>
- Surface: <route / view>
- Viewport: <1280×800 | 375×812 | both>
- Auth: <signed-out | signed-in | either>
- Drive path: <click/type/submit sequence>
- Observed: <result + optional console/network one-liner>
- Screenshot: <screenshots/walk-…/….png or none>
- Coverage unit: <path from coverage.md>
```

## Titles

- Bug: `Fix [broken behavior] on [surface]`
- Improvement: `Align [component] on [surface] with [sibling or token]`
- Idea: `Add [exact control/flow] to [surface]`
- Empty: `Handle empty state for [list] on [surface]`
- A11y: `Fix [specific a11y failure] on [surface]`

No trailing period. No “Polish UI”. No “Walk finding”.

## Epic body

```markdown
## UI walk

- Date: <YYYY-MM-DD>
- Live URL: <url or local preview>
- Surface: <all front-facing | bias>
- Intent: Package solve-ready leaves from a live front-facing UI walk.
- **Do not implement this epic shell.** `/solve` expands to eligible children.

## Leaves

(filled after create)

## Notes

- Filed unassigned in Backlog/Todo for `/solve`.
- Kind mix: bugs, ideas, and improvements from the walk.
```
