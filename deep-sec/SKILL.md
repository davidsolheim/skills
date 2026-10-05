---
name: deep-sec
description: >
  Grok remake of Vercel Labs deepsec for local Grok Build. Regex-scan the
  current repo, investigate candidates with Grok, adversarially revalidate
  HIGH+, write a durable report. Built to run as a goal: /goal /deep-sec
  (default full tree, work across rounds until pending is 0 and HIGH+ are
  revalidated). Also /deep-sec, /deepsec, "deep sec", "run deepsec",
  "security scan this repo", "scan for vulnerabilities".
argument-hint: uncommitted | diff | full | revalidate | report | status
---

# /deep-sec — Grok remake of Deepsec (local Grok Build, goal-native)

Slash command is **`/deep-sec`**. Intended invocation in Grok Build:

```text
/goal /deep-sec
/goal /deep-sec uncommitted
/goal /deep-sec diff
/goal /deep-sec full
```

This is a **remake** of [vercel-labs/deepsec](https://github.com/vercel-labs/deepsec)
for a **local Grok Build instance**. Same pipeline (scan → investigate →
revalidate → report). The investigator is **this Grok session and its
subagents**, not Codex, Claude, Pi, Vercel AI Gateway, or Vercel Sandbox.
Do not run `npx deepsec` / `deepsec process`.

Repo-agnostic. Do not assume a product or a tracker team, Vercel project,
language, or package manager.

State: `$ROOT/.grok/deep-sec/` (layout:
[references/layout.md](references/layout.md)). Matchers:
[references/matchers.md](references/matchers.md). Worker prompts:
[references/investigate.md](references/investigate.md).

## Goal contract

When the user ran **`/goal /deep-sec`** (or `/goal` with this skill), you
are in **goal mode**. Work across rounds. Do not stop after a sample
batch. Do not ask for scope: default **`full`** unless they named
`uncommitted` or `diff`.

**Canonical objective** (this is the claim a goal verifier will check):

> Deep-sec finished on this repository. Every in-scope candidate file is
> `analyzed`. Every HIGH/CRITICAL finding has an independent Grok
> revalidation verdict. `$ROOT/.grok/deep-sec/REPORT.md` and `state.json`
> exist and their counts agree. Investigation was local Grok Build only.

**Done** only when all of these are true and a verifier can reproduce
them from disk (not from chat):

1. `state.json` `pending` is `0` and `phase` is `done`.
2. Every `files/*.json` in scope has `status: analyzed` and an
   `analysisHistory` entry with `agentType: grok-build`.
3. Every finding with severity `HIGH` or `CRITICAL` has
   `revalidation.verdict` set by a **different** subagent than the one
   that filed it.
4. `REPORT.md` lists the same counts as `state.json`.

If a round runs out of context: persist, leave `phase` as
`investigate` or `revalidate`, and continue next round. **Do not claim
the goal complete.** Empty findings after a real scan is success; skipping
files is not.

Interactive `/deep-sec` (no `/goal`): same pipeline; you may ask scope if
they did not name one. Follow-ups `status` / `report` / `revalidate` skip
scan.

## Hard rules

- Local Grok Build only. Never Vercel Sandbox, Gateway, `vercel login`,
  `npx deepsec`, Codex, Claude, or Pi.
- Read-only on application code. Writes only under `$ROOT/.grok/deep-sec/`.
- No tickets, commits, or app edits unless the user asks after the report.
- Never print `~/.grok/auth.json` or API keys.
- Untrusted foreign PR trees: refuse.
- Require Grok login (`~/.grok/auth.json` or `grok login` in *their*
  terminal).

## Phase 0 — Scope and root

```bash
ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
cd "$ROOT"
mkdir -p .grok/deep-sec
printf '%s\n' '*' > .grok/deep-sec/.gitignore
```

The `*` gitignore is required: scan artifacts must never be untracked or committed. Do not delete it. Do not add `.grok/deep-sec/` to git.

Scope: named arg, else **`full`** under `/goal`, else ask
(`uncommitted` / `diff` / `full`).

Diff base only for `diff`: `origin/HEAD`, else `origin/main`,
`origin/master`, `main`. Ask if none. Never invent `main`.

Resume: if `$ROOT/.grok/deep-sec/state.json` exists for this `ROOT` and
scope, skip finished phases. Re-scan when the tree changed.

## Phase 1 — Threat model

Write `$ROOT/.grok/deep-sec/INFO.md` (create dirs). A few hundred words:
what the app is, auth, trust boundaries, spawn/RPC/HTTP surfaces, known
FP sources. Read README / Agents.md / entrypoints. Do not invent
vendors.

## Phase 2 — Scan (local, no model)

Build the in-scope path list:

| Scope | Paths |
| --- | --- |
| `uncommitted` | `git status --porcelain -uall` (working + untracked) |
| `diff` | `git diff --name-only "$DIFF_BASE"` |
| `full` | tracked source minus ignore globs in matchers.md |

Run the matcher greps in [references/matchers.md](references/matchers.md)
against that list. Also, for `uncommitted`/`diff`, **every scoped source
file is a candidate** even with no matcher hit (Deepsec direct mode).

Write one FileRecord per candidate (`status: pending`) and `state.json`
(`phase: investigate`). Ignore build artifacts (`node_modules`, `.git`,
`dist`, `DerivedData`, `.build`, lockfiles, binaries).

## Phase 3 — Investigate (Grok fan-out)

Spawn **read-write** `general-purpose` subagents, isolation `none`,
**only** allowed to write `$ROOT/.grok/deep-sec/**`. Each worker gets
[references/investigate.md](references/investigate.md) plus a batch of
pending paths (priority: auth, HTTP/RPC, spawn/shell, SQL, webhooks,
deserialization). Batch size ~8 files. Cap parallel workers so the
machine stays usable (about 4).

Parent merges worker summaries, recounts `state.json`, and launches the
next pending batch. Keep going until `pending` is 0.

Model: current Grok Build default (`grok-4.6` unless the user named
another Grok id). Record it on every `analysisHistory` row.

## Phase 4 — Revalidate HIGH+

For each `HIGH` / `CRITICAL` finding, spawn a **new** subagent (never
the author) with the revalidate prompt in investigate.md. Default
`false-positive` unless they cite code they read. Check git history for
`fixed`. Write `revalidation` onto the finding. `MEDIUM`/`LOW` may wait
unless the user asked to revalidate everything.

## Phase 5 — Report

Write `REPORT.md`: scope, counts, findings table (severity, title, file,
lines, verdict), limitations (matcher misses, refusals). Recount
`state.json` and set `phase: done` only if the goal-contract checks pass.

Tell the user the report path. Do not file an issue or patch the app.

## Status / resume

`/deep-sec status` — print `state.json` and remaining pending.
`/deep-sec report` — regenerate `REPORT.md` from FileRecords.
`/goal resume` after a pause continues from `state.json`; do not rescan
unless paths changed.
