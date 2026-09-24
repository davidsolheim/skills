# Batch plan format (`/issues`)

Internal + user-facing plan before Linear create. Keep the user table short;
keep the machine fields complete.

---

## User-facing table (required before file)

```markdown
## Plan (N leaves · epic: <title|none>)

| # | Title | P | Class | Package | Links | Action |
|---|-------|---|-------|---------|-------|--------|
| L1 | Add … | 2 | foundation | app | — | create |
| L2 | Fix … | 2 | feature | app | blockedBy L1 | create |
| L3 | Align … | 3 | polish | app | related L2 | create |
| L4 | Old … | 3 | feature | app | dup 0199 | skip |
| L5 | Old modal … | 3 | feature | app | contradicted by L1 | drop-contradicted |
| L6 | Settings page … | 2 | feature | app | retire 0040 | create + retire 0040 |
```

Priority column: `1` Urgent … `4` Low (or High/Medium labels consistently).

---

## Internal record (per leaf)

```text
temp_id: L2
title: …
type: bug|feature|chore|…
priority: 1-4
class: foundation|feature|polish|content|a11y|chore
package: app|services|agents|…
connectivity: independent|related|blocked|epic-child
blocked_by: [L1]
related: [L3, 0210]
duplicate_of: null | 0199
contradicts: [] | [0040, L4]
supersedes: [] | [{ mode, ids, scope, board_action }]
user_fragment: "quoted bullet"
primary_paths: [… from investigation]
```

---

## Epic record (optional)

```text
create_epic: true|false
epic_title: …
epic_reason: shared initiative | --epic flag
children: [L1, L2, L3]   # not skips
```

---

## Caps & deferrals

When `--max N`:

- Sort by class order (foundation first) then priority  
- `action: create` for top N  
- Remainder `action: deferred` with reason `max N`  
- List deferred titles in the final reply  

---

## Stop conditions

| Mode | After plan |
|------|------------|
| `--plan-only` | Stop; no bodies required |
| `--draft` | Write full bodies for create rows; no Linear |
| default | File create rows; skip dups; drop contradicted intra-batch leaves; retire unstarted board contradictions after create; report deferred/failed |
