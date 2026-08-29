# Linear comments — read before write

Before every `*_save_comment` on an issue:

1. `list_comments` on **that** issue (recent first). A list from earlier in
   the run is stale if another worker or a leftover babysit tick may have posted.
2. Use the thread: human direction, blockers, live `claimed-by:` /
   `stolen-by:`, prior closeout or ship notes.
3. **Skip** posting when a comment already covers this **same moment**
   (same first-line marker, heading, PR number, or merge SHA). Do not
   “improve” a match by posting another.
4. Then `*_save_comment` with literal newlines (not `\n` escapes).

Do **not** `list_comments` on every skipped candidate. Only the issue you are
about to comment on or claim.

Skills that post comments (`/prb`, `/yeet`, `/solve`, `/identify`, `/tidy`,
`/issue` retire, `/walk` retire, `/start` nested `/solve`) follow this file.
Moment keys live in each skill. `/start` itself does not comment on V1 leaves
until nested `/solve` claims them.

`/prb` Phase 1.5 posts **one** `/prb — local review gate` comment on the first
ship issue (template C in `prb/references/linear-ship-comments.md`). It does
**not** file Linear issues per finding. `/solve` posts claim + one In Review
closeout; it does not post per-reviewer chatter.

| Excuse | Reality |
|--------|---------|
| "I already loaded the issue body" | Body ≠ comments. List comments. |
| "We posted this earlier in the run" | Re-list. Orchestrator and scheduler race. |
| "A second comment will be clearer" | Skip. One comment per moment. |
| "File a Linear issue per /prb finding" | No. Scratch markdown + one gate comment. |
| "This is a different skill, so a new comment is fine" | Same moment + same PR/SHA/marker → skip. |
