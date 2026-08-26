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
`/issue` retire) follow this file. Moment keys live in each skill.

| Excuse | Reality |
|--------|---------|
| "I already loaded the issue body" | Body ≠ comments. List comments. |
| "We posted this earlier in the run" | Re-list. Orchestrator and scheduler race. |
| "A second comment will be clearer" | Skip. One comment per moment. |
| "This is a different skill, so a new comment is fine" | Same moment + same PR/SHA/marker → skip. |
