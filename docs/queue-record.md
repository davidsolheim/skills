# Queue record

The issue file under `.wcp/issues/` is the record. Notion mirrors it after the file write (`notion-issues.md`). Do not call Linear. Do not post tracker comments.

`/prb` and `/yeet` record Dev SHA, the PR URL, and Main SHA on the issue file and the Notion row (`../prb/references/ship-comments.md`). `/tidy` writes a `tidy-pass:` line in the issue file (`../tidy/references/ledger.md`). `/solve` sets the file status. It does not set Notion `done`.

| Excuse | What to do |
|--------|------------|
| "File a Linear issue per /prb finding" | No. Scratch markdown is the gate record. |
| "Post a claim comment" | No. Claim is the wcp ticket lease. |
| "Post a second note for the same ship" | No. One Dev SHA and one Main SHA on the row. |
