# `/start` handoff

Emit after Phase 6 (or earlier stop). Do not start `/prb` unless asked.

```markdown
**Started:** <Product name>
**Mode:** greenfield | existing
**Path:** `<DEST>`
**Bones:** pass | fail <list>   <!-- existing: required; greenfield: n/a after scaffold -->
**Docs/identity:** pass | repaired <list>
**Slug / Doppler:** `<slug>` / `<slug>` (project may still need creating in Doppler)
**Git:** local `main` + `dev` · origin: <url or none> · push: none (default)
**Docs:** `AGENTS.md` · `VISION.md` · `README.md` · `.linear-project`
**Linear:** <team> / [project](url) · epic [TEAM-123](url)
**V1 leaves:** N filed · K thin/skipped
**Build:** `/solve all` [fast] · solved S · failed F · drain: <verified | not run | blocked>
**Verify:** <matrix commands + pass/fail> · runtime: <driven V1 path or blocked: missing DATABASE_URL/Doppler>
**Next:** `/prb` when you want `dev` → main · create Doppler/Neon if named as blockers

### V1
| ID | Title | Linear | Notes |
|----|-------|--------|-------|
| [TEAM-123](url) | … | In Review | …

### Blockers
- <none | Doppler project | Neon DATABASE_URL | Linear auth | dest conflict | bones fail>
```

Omit empty Blockers. If `--docs-only` / `--draft` / `--no-build`, say so in
**Build** (`not run — flag`).

Point the user at DEST. They should reopen Grok **in that repo** for later
`/solve` / `/issue` so AGENTS Linear binding resolves.
