# Vault chrome (Obsidian)

Display names, home Index, catalog, tags, aliases, graph colors, Dataview.
Note body shape stays in [`templates.md`](templates.md). Prose quality stays
in [`leapfrog.md`](leapfrog.md).

## Display names

Never lead with a git slug when a product name exists. Order:

1. README / VISION first `#` heading (strip “Welcome to”)
2. `package.json` `"name"` if it is not `@scope/repo-slug`
3. Title-case the slug (`ore-max` → Ore-Max, `angela-solheim-com` → Angela Solheim)

Use that string as the note title alias and as wikilink aliases:

```yaml
aliases: [Ore-Max]
```

Home Index links: `[[projects/ore-max/Index|Ore-Max]]`.

## Tags (frontmatter)

Project feature:

```yaml
tags:
  - status/active   # or status/stale
  - domain/auth     # see domains below
```

Pattern: `type/pattern` plus the same `domain/…`.
Project Index: `type/project`.
Empty project (zero active features): also `status/empty`.

**Domains** (pick one):

| Domain | Canonicals |
|--------|------------|
| `auth` | Auth, Impersonation, SSO, Magic link, Password reset, Two-factor authentication, Profile, Team invites, Roles and permissions, Organizations |
| `commerce` | Billing, Checkout, Subscriptions, Invoices, Orders, Products, Cart, Loyalty, Donate |
| `content` | CMS, Blog, Gallery, Media, Compose, Posts, Reports |
| `ops` | Audit log, Feature flags, Background jobs, Scheduled tasks, Webhooks, Public API, API keys, Import and export, Admin, Dashboard |
| `growth` | Analytics, Referrals, Waitlist, Newsletter, Feedback, Search |
| `site` | Homepage, About, Contact, Legal, SEO, Localization |
| `other` | everything else |

## Home Index (`Index.md`)

Not a flat slug list. Required sections, in order:

1. One-line what this vault is
2. **Start here** — 4–8 pattern hubs that exist (Auth, Impersonation, Billing, Checkout, File uploads, Team invites, Magic link, SSO). Skip missing.
3. **Products** — apps with Auth or Billing or a distinct operator console
4. **Sites** — brochure/marketing (Homepage/About/Contact/Legal dominate)
5. **Tools** — CLI, scripts, small utilities
6. **Empty** — zero active features (collapsed; do not omit the link)
7. **Patterns** — link `[[patterns/Index]]` and `[[catalog]]`

## Catalog (`catalog.md`)

A **list of pattern notes**, not a table with 40 wikilinks in one cell.

```markdown
- [[patterns/Auth|Auth]] — 49 apps · login, session, authentication
```

The project list lives on the pattern page. Optional Dataview (ignore if the
plugin is off):

````markdown
```dataview
TABLE length(rows) AS Apps
FROM "projects"
WHERE type = "project-feature" AND status = "active"
GROUP BY canonical
SORT length(rows) DESC
```
````

Unused starter canonicals may remain as one line with **0 apps** and no
pattern link.

## Graph

Seed `<vault>/.obsidian/graph.json` `colorGroups` if missing or empty:

- query `path:patterns` — warm (pattern hubs)
- query `path:projects` — cool (instances)
- query `tag:#status/stale` — muted
- query `tag:#status/empty` — muted

Do not launch Obsidian. Local graph on a **pattern** note is the leapfrog view.

Show tags in graph (`showTags: false` is fine; color groups matter more).

## Connections vs graph

Only `depends_on`, `unlocks`, and `[[patterns/Canonical]]`. Extra sibling
links turn the graph into a hairball.
