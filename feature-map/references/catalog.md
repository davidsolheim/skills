# Catalog matching

Vault `catalog.md` is the source of truth for canonical names and aliases
after first run. This file is the matching rules plus the **starter table**
copied into a new vault.

## Granularity

Use the feature definition in `SKILL.md` § Operating contract. Do not fold a
capability into a sibling because it requires that sibling.

## Match algorithm

1. Read vault `catalog.md`.
2. Normalize a candidate: lowercase, trim, hyphens/spaces equivalent.
3. If it equals a canonical or any alias → **known**. Use that canonical.
4. Else if it clearly matches a starter-table alias and that canonical is
   already in the vault catalog → **known**.
5. Else if it clearly matches a starter-table row not yet in the vault →
   **propose** that canonical (pre-filled name). Still list it in the
   confirmation table so the user can rename.
6. Else → **unknown**. Propose a Title-Case name (noun, short). Suggested
   aliases = names you actually saw in code.

Never attach the same alias to two canonicals. If the table would, **ask**.

After confirmation, write vault `catalog.md`: every canonical used in this
run, union of aliases, pattern wikilink, project wikilinks.

## Confirmation table (once per project)

Wait before writing notes.

| Proposed canonical | Why | Suggested aliases | Sample paths |
|--------------------|-----|-------------------|--------------|

- User accepts → use it.
- User renames → their name is canonical; your proposal becomes an alias.
- User says two proposals are the same feature → one canonical, merge aliases.
- User says split further → split. Do not argue.

Known matches do **not** appear in this table.

## Vault `catalog.md` shape

```markdown
---
type: catalog
updated: YYYY-MM-DD
---

# Feature catalog

Canonical names and aliases. Updated by `/feature-map` after confirmation.

| Canonical | Aliases | Pattern | Projects |
|-----------|---------|---------|----------|
| Impersonation | login-as, act-as, masquerade | [[patterns/Impersonation]] | [[projects/ore-max/Impersonation]] |
```

## Starter table (seed + proposal hints)

| Canonical | Aliases |
|-----------|---------|
| Auth | login, log-in, sign-in, signin, sign-on, session, authentication |
| Impersonation | impersonate, impersonation, login-as, log-in-as, act-as, masquerade, spoof-user |
| Magic link | magic-link, email-link-login, passwordless-email |
| Two-factor authentication | 2fa, totp, mfa, two-factor, otp |
| SSO | sso, saml, oidc, oauth-sso |
| Password reset | password-reset, forgot-password, recover-password |
| Onboarding | onboarding, first-run, getting-started-flow |
| Team invites | invites, invitations, invite-link, seat-invite |
| Roles and permissions | rbac, roles, permissions, authorization |
| Organizations | orgs, workspaces, tenants, teams-account |
| Billing | billing, customer-portal, payment-method |
| Checkout | checkout, paywall-checkout, collect-payment |
| Subscriptions | subscriptions, recurrences, plans |
| Invoices | invoices, invoicing |
| Audit log | audit-log, activity-log, admin-audit |
| Feature flags | feature-flags, flags, kill-switch |
| File uploads | uploads, attachments, file-picker |
| Transactional email | transactional-email, mailer, sendgrid, resend, postmark |
| Notifications | notifications, in-app-alerts, notification-center |
| Search | search, full-text-search, command-palette-search |
| Analytics | analytics, product-analytics, tracking-events |
| Feedback | feedback, feedback-widget, user-feedback |
| Webhooks | webhooks, outbound-webhooks, webhook-endpoints |
| Public API | public-api, rest-api, partner-api |
| API keys | api-keys, pats, personal-access-tokens |
| Background jobs | background-jobs, queues, workers, sidekiq, bullmq |
| Scheduled tasks | cron, scheduled-jobs, heartbeat |
| Import and export | csv-import, csv-export, data-export, gdpr-export |
| Waitlist | waitlist, access-request |
| Referrals | referrals, affiliate-links |
| Profile | profile, account-settings |

First-run copies this table into the vault with empty Pattern/Projects. Later
runs fill Pattern/Projects for features that appear; they do not delete unused
starter rows.
