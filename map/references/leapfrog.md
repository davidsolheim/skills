# Leapfrog writing

Thoroughness beats brevity. Read the **key source files** for a feature before
writing a word. Token cost is not a constraint.

A note is a failure if someone about to build the same capability in a new
app cannot tell **what to copy and what to skip**.

## Project feature notes

**What it is** — 3–8 sentences about *this capability in this app*. Who uses
it (customer, operator, both). What they can do. What it is *not* (no public
signup, no OAuth, admin-only, etc.). Do **not** paste the README, the stack
list, or “is a capability you could ship or copy on its own.”

**How it works** — the real flow, named. Which provider (Auth.js credentials,
Clerk, Stripe Checkout Session, Resend). Where the session/row lives. What
happens on success and failure. Cite files as `` `path` `` after you have
read them.

**Sequence** — mermaid that matches **this** flow (actors and steps you
actually saw). A generic `User → Feature → Code` flowchart is a failed note.

**Connections** — only:

- **Depends on:** hard prerequisites that exist in this project (`[[Auth]]`)
- **Unlocks:** capabilities in this project that require this one
- **Pattern:** `[[patterns/Canonical]]`

Do not link every sibling. Graph edges should mean something.

**Surfaces** — real routes, UI entry, APIs. Not a guess.

**Implementation map** — 3–8 files that matter, with a **specific** role
(“Auth.js config + credentials authorize”, not “Implementation”). Drop
wishlist-session-next-to-auth noise.

**Data and config** — models/tables and env **names** you saw. Never values.

**Gotchas** — what would burn someone porting it: adapter table names, JWT vs
DB sessions, webhook signature, locale prefix, impersonation not in Auth.

**Port to a new app** — a recipe a future `/start` could follow:

1. Prerequisites (other features)
2. Stack this recipe assumes (named vendors)
3. Recreate — ordered files/concepts, not “port the map”
4. Env names
5. Verify — signed-out, signed-in, one failure; name the URL
6. Do not copy — secrets, brand copy, one-off schema

## Pattern notes

This page is the leapfrog surface. It is **not** a dump of project blurbs.

**Opening** — portable definition of the capability (two to four sentences).
Not “a capability you could ship.”

**Across projects** table columns (required):

| Column | Content |
|--------|---------|
| Project | Display name, linked |
| Mechanism | Provider + where state lives (one line, specific) |
| Steal | What is actually worth copying |
| Skip | What is local/hacky |
| Note | `[[projects/<slug>/Canonical]]` |

No identical rows. If two apps use the same Auth.js + Neon pattern, say so
once in Steal and still fill Mechanism from *that* repo’s files.

**Default recipe** — name **one** donor project (`[[projects/ore-max/Auth]]`)
and why (clearest files, fewest hacks). Then three to six steps. Do not say
“follow whichever row you clicked.”

**Aliases** — from the catalog.

## Forbidden filler

- “is a capability you could ship or copy on its own”
- “Related capabilities in this app are linked under Connections”
- “Implementation map and port recipe” / “Repo-specific secrets and copy”
- “Direct dependency `foo` in this app” as Stack *why*
- Generic mermaid `User -->|uses| Feature`
- README / VISION pasted into What it is
