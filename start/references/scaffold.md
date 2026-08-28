# Scaffold from next-starter-template

Used by `/start` Phase 1–2. Do not run this against the template repo itself.

## Where

| Condition | DEST |
|-----------|------|
| `--dir PATH` | that absolute path |
| else | `$HOME/src/<slug>` |
| cwd is empty, not `$HOME`, not the template, and user did not pass `--dir` | cwd (still name the slug) |

**Refuse (this file is greenfield only):**

- DEST is, or is inside, `next-starter-template`
- slug is `next-starter-template`

If DEST exists and is **non-empty**, do not scaffold: the skill sets
`MODE=existing` and uses [`existing-repo.md`](existing-repo.md) instead.

Create parent dirs as needed. Do not scaffold into `$HOME` itself.

## Source

Prefer a clean GitHub clone over a dirty local tree:

```bash
SRC_REMOTE=https://github.com/davidsolheim/next-starter-template.git
```

If a local clone of that repo exists (common: `$HOME/src/next-starter-template`)
**and** `git status` there is clean, it may be used as a file source. If it is
dirty or missing, clone from `SRC_REMOTE` into a temp dir.

## Copy (no template history)

```bash
mkdir -p DEST
# from SOURCE (cloned or local):
rsync -a \
  --exclude '.git' \
  --exclude 'node_modules' \
  --exclude '.next' \
  --exclude 'tsconfig.tsbuildinfo' \
  --exclude '.DS_Store' \
  SOURCE/ DEST/
```

Keep `bun.lock`, `drizzle/`, `docs/`, `tests/`, starter `AGENTS.md` (first-run marker intact until Phase 3 overwrites it).

Then:

```bash
cd DEST
git init
git checkout -b main
```

Do **not** `git clone` and keep `.git`. Do **not** add the template as `origin`.

## Identity retarget (before first commit)

In DEST, replace starter identity:

| File | Change |
|------|--------|
| `package.json` `name` | `<slug>` |
| `package.json` `repository` / `bugs` / `homepage` | this repo if GitHub known; else delete template URLs |
| `package.json` `scripts.doppler:setup` | `--project <slug>` |
| `doppler.yaml` `setup.project` | `<slug>` |
| `doppler.yaml` `setup.config` | `development` (keep) |

Do not rewrite product copy yet (Phase 3). Do not invent a Doppler project on Doppler’s cloud here.

## First commit

```bash
git add -A
git status   # confirm no .env with values, no node_modules
git commit -m "$(cat <<'EOF'
chore: scaffold from next-starter-template

EOF
)"
```

LICENSE stays MIT © David Solheim (starter). Product README will credit it.

## Branches

After Phase 3 docs commit (or immediately if docs land in the same commit):

```bash
git checkout -b dev main   # only if dev does not exist
# if docs commit landed on main after dev existed:
git checkout dev && git merge main
```

Integration branch is lowercase **`dev`** only.

## GitHub remote (optional)

Create `$GH_USER/<slug>` **private** only when:

- user asked for a GitHub repo, or
- BRIEF/`--dir` already names `github.com/<owner>/<slug>`

```bash
GH_USER=$(gh api user --jq .login)
# if BRIEF named github.com/<owner>/<slug>, use that owner instead
gh repo create "$GH_USER/<slug>" --private --source=. --remote=origin --push
```

Default: no `origin`. Never `--public` unless the user said public. Never force-push.

## Sanity check (required)

- [ ] DEST ≠ template path
- [ ] `git remote -v` does not list next-starter-template
- [ ] `package.json` name is `<slug>`
- [ ] `doppler.yaml` project is `<slug>`
- [ ] `git log` has no template authors/commits
- [ ] `main` exists; `dev` exists after onboard commit
