# Workspace layout and schemas

Root: `$REPO/.grok/deep-sec/`

```
.gitignore                # `*` — never track this directory
INFO.md
state.json
REPORT.md
files/<safe-path>.json    # one FileRecord; `/` in relpath → `__`
findings/<slug>.md        # one human file per finding
```

`safe-path` = repo-relative path with `/` replaced by `__`. Example:
`FuturaTerm/System/RemoteSpawn.swift` →
`files/FuturaTerm__System__RemoteSpawn.swift.json`.

Writes stay under this directory. Application source is read-only.

## `state.json`

```json
{
  "root": "/abs/path",
  "scope": "full",
  "diff_base": null,
  "model": "grok-4.6",
  "phase": "scan",
  "candidates": 0,
  "analyzed": 0,
  "pending": 0,
  "findings": 0,
  "high_plus": 0,
  "revalidated": 0
}
```

`phase`: `scan` | `investigate` | `revalidate` | `report` | `done`.
`diff_base` is a git ref or `null`. Recount from FileRecords before
every persist. Goal-complete requires `phase=done` and `pending=0`.

## FileRecord (`files/*.json`)

| Field | Type |
| --- | --- |
| `filePath` | repo-relative path |
| `candidates` | `{ vulnSlug, lineNumbers, snippet }[]` |
| `findings` | Finding[] |
| `analysisHistory` | AnalysisEntry[] |
| `status` | `pending` \| `analyzed` \| `error` |
| `fileHash` | sha-256 of source at last scan (optional) |

Finding:

| Field | Type |
| --- | --- |
| `severity` | `CRITICAL` \| `HIGH` \| `MEDIUM` \| `HIGH_BUG` \| `BUG` \| `LOW` |
| `vulnSlug` | matcher slug or `other-<topic>` |
| `title` | one sentence |
| `description` | full why |
| `lineNumbers` | 1-indexed |
| `recommendation` | suggested fix |
| `confidence` | `high` \| `medium` \| `low` |
| `revalidation` | optional Revalidation |

AnalysisEntry (append):

| Field | Notes |
| --- | --- |
| `runId` | `grok-<YYYYMMDDHHMMSS>-<4 hex>` |
| `investigatedAt` | ISO |
| `agentType` | `grok-build` |
| `model` | e.g. `grok-4.6` |
| `findingCount` | integer |

Revalidation (HIGH+ only, different agent):

| Field | Type |
| --- | --- |
| `verdict` | `true-positive` \| `false-positive` \| `fixed` \| `uncertain` |
| `reasoning` | cite code; git evidence when `fixed` |
| `revalidatedAt` | ISO |
| `model` | Grok id |

## Goal evidence

A verifier must be able to confirm, using only this directory:

```bash
python3 -c "
import json, pathlib
d=pathlib.Path('.grok/deep-sec')
st=json.loads((d/'state.json').read_text())
files=list((d/'files').glob('*.json'))
pending=sum(1 for p in files if json.loads(p.read_text()).get('status')!='analyzed')
print('phase', st.get('phase'), 'pending_disk', pending, 'pending_state', st.get('pending'))
print('report', (d/'REPORT.md').exists())
"
```

Mismatch between disk and `state.json` → goal not complete.
