# /prb — Project ship product build (apps / installers)

Some repos ship a **built product** (macOS `.app`, signed binary, etc.) that must be regenerated when code changes. `/prb` discovers and runs **this repo’s** documented ship-build command so the artifact matches the commits being pushed.

This is **not** CI. It is a local pre-push (and post-fix) rebuild of the operator-facing product when the project opts in via `AGENTS.md` (or equivalent).

---

## 1. Detect whether this project has a ship product build

Read **in order**:

1. Root `AGENTS.md` / `CLAUDE.md` sections titled like:
   - `## /prb — ship product build`
   - `## Ship product build`
   - `## Release build`
2. `README.md` “Ship” / “Release” / “macOS app” sections that name a script under `scripts/`
3. Executable scripts: `scripts/build-mac-app.sh`, `scripts/ship-*.sh` **only if** AGENTS/README says `/prb` must run them

### Discovery table

| Field | How to resolve |
|-------|----------------|
| `SHIP_BUILD_REQUIRED` | `yes` if AGENTS documents a required `/prb` ship build for this repo; else `no` |
| `SHIP_BUILD_CMD` | Exact command (e.g. `bash scripts/build-mac-app.sh`) |
| `SHIP_BUILD_OUTPUT` | Path of artifact (e.g. `dist/LeetBridgeMac.app`) — local only unless docs say commit it |
| `SHIP_BUILD_TRIGGER` | Paths that force rebuild (default: `Sources/**`, `Apps/**`, `Package.swift`, `*.xcodeproj/**`, the build script) |
| `SHIP_BUILD_SKIP` | User passed `--skip-ship-build` → skip with loud warning |

If `SHIP_BUILD_REQUIRED=no`, skip this reference (report `Ship build: n/a`).

---

## 2. When to run during `/prb`

| Step | When | Action |
|------|------|--------|
| Inventory | Phase 0 | Fill discovery table from AGENTS |
| Pre-push | After Phase 1.5 **clean** (or `--skip-review`), **before** Phase 1D `git push origin dev` | Run `SHIP_BUILD_CMD` when required |
| Babysit fix | After re-review clean, **before** re-pushing `dev` | Re-run if ship set still matches trigger paths |
| Failure | Non-zero exit | **Do not push**; report build log tail |

**Default for required projects (e.g. LeetBridge):** always rebuild when shipping commits on `dev`, even for small diffs, unless `--skip-ship-build`.

Do **not** commit `dist/` unless the user explicitly asks and `.gitignore` is adjusted.

---

## 3. Execution checklist

1. Working tree: no unresolved conflicts; on local `dev` after main merge + review gate.
2. Run from git root:
   ```bash
   bash scripts/build-mac-app.sh   # example — use SHIP_BUILD_CMD from discovery
   ```
3. Generous timeout (Release + codesign can take several minutes).
4. On success: note timestamp + output path in the session report.
5. On failure: capture redacted tail of the log; **block push**.
6. Never print signing passwords or notary credentials. Prefer identities already in the keychain.

### Skip

- `--skip-ship-build` only with explicit user intent.
- Project has no ship-build section → skip silently (`n/a`).

---

## 4. Report fields (append to Phase 5)

```markdown
**Ship build:** n/a | rebuilt `dist/…` @ <time> | skipped (--skip-ship-build) | blocked (<reason>)
**Ship command:** `<SHIP_BUILD_CMD>` | n/a
```
