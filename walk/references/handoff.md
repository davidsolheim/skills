# /walk handoff

Scannable. Full bodies live in Linear (or scratch `final/` if not filed).

---

## Filed

```markdown
**UI walk complete**
- **Team / project:** <team> / <project>
- **Live URL:** <url>
- **Auth:** public walked | signed-in walked | blocked_auth on <n> units
- **Filed:** yes

### Epic
- [<ID>](url) — UI walk – <Project> – <YYYY-MM-DD>
  (or none)

### Coverage
| Status | Count |
|--------|-------|
| walked | n |
| blocked_auth | n |
| blocked_flag | n |
| broken_load | n |
| not_front | n |
| pending (must be 0) | n |

Units not walked: <none | list>

### Counts
| | Bug | Improvement | Idea | Total |
|--|-----|-------------|------|-------|
| Filed | n | n | n | n |
| Skipped duplicate | n | | | |
| Thin (not filed) | n | | | |

### Unblocked leaves (ready for `/solve`)
1. [TEAM-101](url) — <title> · bug · P0
2. …

### Suggested next
/identify
/solve

### Duplicates skipped
- …

### Retired
- …

### Conflict — needs you
- …

### Scratch
- **Status:** deleted after successful Linear file **or** kept at `<abs path>`
```

---

## Draft / Linear unavailable

```markdown
**UI walk complete — NOT FILED**
- **Reason:** `--draft` | Linear failed: <brief>
- **Scratch (keep):** `<abs path>`
- **Finals:** `…/issue-candidates/final/` (N)
- **Coverage:** walked n / blocked_auth n / pending 0
```

---

## Partial file

```markdown
**UI walk partial file**
- Created: [ids]
- Failed: [titles]
- **Scratch kept:** `<abs path>`
- Re-file remaining `final/` — do not re-walk unless coverage was incomplete
```

---

## Tone

- Do not paste every coverage row unless some are not `walked`
- Do not start `/solve` unless asked
- Do not claim “whole product reviewed” — this was front-facing UI only
