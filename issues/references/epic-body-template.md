# Epic body template (`/issues` parent)

Epics are **packaging only**. Never put the only acceptance criteria on the
epic. `/solve` and cheap-model handoffs expand to eligible children and must
not implement the shell. All execution depth lives on **child** tickets
([`../../issue/references/execution-ready-bar.md`](../../issue/references/execution-ready-bar.md)).

```markdown
## Initiative

- Source: `/issues` batch — <one-line theme from user dump>
- Date: <YYYY-MM-DD>
- Intent: Package related solve-ready leaves from a multi-item intake.
- **Do not implement this epic shell.** Expand to children.

## Scope

- In: <what this cluster covers>
- Out: <explicit non-goals / other packages>

## Leaves

(filled after create, or leave empty — children are the work)

- TEAM-… — …
- TEAM-… — …

## Dependency notes

- Hard deps use child `blockedBy` (not epic state).
- Independent chores from this dump were filed **flat** (not under this epic)
  when connectivity was weak.

## Notes

- Filed unassigned in Backlog/Todo for `/solve`.
- Batch plan title: <…>
```
