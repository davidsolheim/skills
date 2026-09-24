# vercel flags CLI cheatsheet

Source: https://vercel.com/docs/cli/flags

Environments: `production` | `preview` | `development` (`-e`).
Prefer `--message` on mutating commands. Ask before production changes.

## list

```bash
vercel flags list
vercel flags ls --state archived
vercel flags list --json
vercel flags inspect welcome-message
vercel flags open welcome-message
```

## create

```bash
vercel flags create my-feature --description "Controls the new onboarding flow"
vercel flags create welcome-message --kind string --description "Homepage welcome copy" \
  --variant control="Welcome back" --variant treatment="Start for free"
vercel flags create layout-config --kind json \
  --variant '{"theme":"light","sidebar":false}'=Light \
  --variant '{"theme":"dark","sidebar":true}'=Dark
```

## enable / disable / set

```bash
vercel flags enable my-feature --environment production --message "Resume rollout"
vercel flags disable my-feature -e production --variant false \
  --message "Pause rollout in production"
vercel flags set welcome-message --environment preview --variant control \
  --message "Serve the control copy in preview"
vercel flags update welcome-message --variant control --value welcome-back \
  --label "Welcome back" --message "Refresh control copy"
```

## rules

```bash
vercel flags rules ls my-feature --environment production
vercel flags rules add my-feature --environment production \
  --condition user.plan:eq:pro --variant on --message "Enable Pro users"
vercel flags rules add my-feature --environment production \
  --condition "user.plan:eq:pro;team.tier:eq:enterprise" --variant on
vercel flags rules update my-feature rule_123 --environment production \
  --condition user.plan:eq:enterprise
vercel flags rules move my-feature rule_123 --environment production --position 1
vercel flags rules rm my-feature rule_123 --environment production
```

## segments

```bash
vercel flags segments ls
vercel flags segments inspect beta-users --json
vercel flags segments create beta-users --label "Beta users" \
  --add include:user.id=user_123 --add include:user.id=user_456
vercel flags segments create enterprise-users --label "Enterprise users" \
  --add rule:user.plan:eq:enterprise
vercel flags segments update beta-users --add include:user.id=user_789 \
  --remove include:user.id=user_123
vercel flags segments rm beta-users --yes
```

## rollout

```bash
vercel flags rollout redesigned-checkout --environment production --by user.id \
  --stage 5,6h --stage 10,6h --stage 25,12h --stage 50,1d \
  --message "Start redesigned checkout rollout"
vercel flags rollout welcome-message --environment production --by user.id \
  --from-variant control --to-variant treatment --default-variant control \
  --stage 10,2h --stage 50,12h --start 2026-04-16T09:00:00Z
```

## split

```bash
vercel flags split ai-summary-model --environment production --by user.id \
  --default-variant stable --weight stable=95 --weight candidate=5 \
  --message "Route summary traffic to the candidate model"
vercel flags split ai-chat-model -e preview --by user.id \
  --default-variant stable --weight stable=50 --weight candidate=50 --weight legacy=0
```

## archive / rm

```bash
vercel flags archive my-feature --yes
vercel flags unarchive my-feature --yes
vercel flags rm my-feature --yes   # must be archived first
```

## sdk-keys

```bash
vercel flags sdk-keys ls
vercel flags sdk-keys add --type server --environment production
vercel flags sdk-keys rm [hash-key]
# Full SDK key is shown only at creation — store securely; never commit.
```

## override

Requires `FLAGS_SECRET` in env / `.env.local` (from `vercel env pull`).

```bash
vercel flags override my-flag=true
vercel flags override flag-a=true flag-b=hello
vercel flags override my-flag=42 --expiration 30d
vercel flags override --decrypt <token>
```

## prepare

```bash
vercel flags prepare
# Writes synthetic @vercel/flags-definitions into node_modules for build-time
# embedding. Usually run automatically by the build when SDK packages/keys present.
```

## evaluations

```bash
vercel flags evaluations new-checkout --since 1h --granularity 15m
vercel flags evaluations new-checkout --since 24h
vercel flags evaluations new-checkout --since 7d --granularity 4h
vercel flags evaluations new-checkout --since 1h --granularity 15m --json
```

## versions

```bash
vercel flags versions welcome-message
vercel flags versions welcome-message --environment production
vercel flags versions welcome-message --limit 10
vercel flags versions welcome-message --limit 10 --cursor next_page_cursor
vercel flags versions welcome-message --json
vercel flags versions diff welcome-message --revision 4
vercel flags versions diff welcome-message --revision 4 --json
```

## Rule operators (quick)

`eq`, `!eq`, `oneOf`, `!oneOf`, `contains`, `!contains`, `startsWith`,
`endsWith`, `containsAllOf`, `containsAnyOf`, `containsNoneOf`, `ex`, `!ex`,
`gt`, `gte`, `lt`, `lte`. Form: `ENTITY.ATTRIBUTE:OPERATOR:VALUE` or
`segment:OPERATOR:SEGMENT`.

