# Execution-ready quality bar

Canonical bar for `/issue`, `/issues`, `/start` V1 leaves, and `/project-review` leaves. Do not
fork this file.

Issues filed are **implementation contracts** for a **weaker / cheaper model**
(Cursor Auto, fast agents, low-effort implementers). Grok does the research;
the ticket must contain almost everything the implementer needs so they only
**verify drift + apply the plan**.

If a cheaper model would need to re-discover architecture, invent an approach,
or guess which files to touch — the ticket is **not ready**.

---

## North star

> A competent junior engineer (or cheap coding agent) who has **never seen this
> repo before** can complete the ticket using **only** the Linear description +
> a light drift check of listed paths — without product discovery, architecture
> redesign, or open-ended investigation.

---

## Required depth (every create leaf)

| Layer | Minimum bar |
|-------|-------------|
| **Where** | Real paths that exist now; primary package/app; symbol names; ~line ranges |
| **What today** | Current behavior with code evidence (not “it seems broken”) |
| **What done looks like** | Checklist AC that a stranger can pass/fail without asking you |
| **How** | Ordered step-by-step plan + file-by-file change list (not only a hypothesis) |
| **Mirror** | At least one existing pattern/file to copy (path + what to copy) |
| **Contracts** | Types, props, schemas, API shapes, env **names** the change touches |
| **Excerpts** | Short code anchors for non-obvious logic (not whole files) |
| **Boundaries** | Explicit out-of-scope / do-not-touch list |
| **Verify** | Copy-paste commands from this repo’s real scripts + **Runtime proof** (drive path, visual reference, blast-radius fact) when in-scope |
| **Drift** | 3–7 anchors; if they match, implementer skips full re-research |
| **Decisions** | Assumptions and ambiguity resolutions already made in the ticket |

---

## Cheap-model failure modes (prevent these in the ticket)

| If the implementer might… | Ticket must instead… |
|---------------------------|----------------------|
| Re-architect “a cleaner way” | State **recommended approach is mandatory** unless drift proves impossible |
| Touch unrelated files | **Do not touch** list + out-of-scope bullets |
| Invent new APIs/components | Point to **reuse / mirror** path and name the symbol |
| Misread product intent | **Expected behavior** as concrete UI/API outcomes |
| Skip edge cases | AC + test plan covering empty/error/auth/mobile if relevant |
| Run wrong checks | Exact package scripts from `AGENTS.md` / `package.json` |
| Stop at typecheck for UI/auth/API | **Runtime proof** section: path to drive + observed end state |
| Expand into a rewrite | “Smallest complete change”; list non-goals |
| Guess stack (Neon vs old DB, etc.) | **Platform / stack** section filled |
| Implement the old direction and the new one | Direction-conflict search; **retire** unstarted contradicted issues |
| Block on a decision | **Assumptions / pre-decided** section so work proceeds |

---

## Investigation depth (intake agent)

Before filing, the intake agent must have **read** (not only grepped) the
primary files enough to write:

1. Accurate current behavior  
2. A credible ordered implementation plan  
3. Real symbol names and contracts  
4. At least one mirror pattern  
5. Verification commands that exist in this repo  

Optional but preferred when cheap:

- 5–40 line **excerpts** of the critical branch/handler (trimmed)  
- Prop/type signatures for components/APIs being edited  
- Example request/response or UI state shapes  

Do **not** dump multi-hundred-line files into Linear. Excerpt only what the
implementer would otherwise re-hunt.

---

## Atomicity (cheap models hate mega-tickets)

One leaf = one shippable outcome. If a cheap model would need a multi-day plan
or many independent packages, **split**. A ticket that lists “also clean up X
and migrate Y” will be under-implemented or over-scoped.

---

## Thin ticket = fail the create gate

Do **not** create when any of these are true:

- No code map with paths that exist in the workspace  
- Acceptance criteria are only “make it better” / “fix the bug”  
- Implementation notes are empty or pure product prose  
- No verification commands  
- No drift anchors  
- Title is vague (“Fix bug”, “Update page”)  

Instead: investigate more, or output a draft in chat / `--draft` and say what
is still unknown — only create when the contract is complete.

---

## Batch / review leaves

Shared research can be holistic, but **each leaf body must still be
self-contained**. Do not write “see L1 / see epic / see sibling” as a
substitute for a code map — a cheap model may be handed only one ticket.

---

## Length guidance

- Prefer **complete** over short. Cheap models fail more often on missing detail
  than on long tickets.  
- Typical solid ticket: **roughly 80–250 lines** of markdown body depending on
  complexity.  
- Epic shells stay short; depth lives on **children**.  
- Secrets never appear (env **names** only).
