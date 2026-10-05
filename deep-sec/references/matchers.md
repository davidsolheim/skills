# Local scan matchers

Always-on greps, inspired by vercel-labs/deepsec's surface classes
(`[CORS][JWT][WEBHOOK][REDIRECT][SQL][AUTH][TRAVERSAL][XSS][CRYPTO][SECRETS][RCE][SSRF]`).
Case-insensitive. Skip ignored paths.

## Ignore

`.git/`, `node_modules/`, `dist/`, `build/`, `.build/`, `DerivedData/`,
`.next/`, `vendor/`, `Pods/`, `target/`, `.grok/`, coverage/cache dirs,
lockfiles, `*.min.js`, images, fonts, binaries, `*.pbxproj`, `*.xcuserstate`.

## Patterns

Run `rg -n -i -g '!{ignore}'` (or equivalent) per slug. A file is a
candidate if any slug hits. Record `lineNumbers` and a short `snippet`.

| vulnSlug | Look for |
| --- | --- |
| `secrets` | `api[_-]?key`, `secret_key`, `BEGIN PRIVATE KEY`, `AKIA[0-9A-Z]{16}`, `xai-`, `sk-`, `ghp_`, password assignments in source |
| `sql` | `execute(` / `query(` / `raw(` with string concat or f-string/interpolation near SQL keywords |
| `command-injection` | `Process(`, `posix_spawn`, `system(`, `exec(`, `popen`, `child_process`, `/bin/sh`, `bash -c` |
| `ssrf` | `fetch(` / `URLSession` / `curl` / `http.get` with user/request input in the URL |
| `path-traversal` | `../`, `path.join`/`appendingPathComponent` with request/filename input, `sendFile`, `readFile` of user paths |
| `xss` | `innerHTML`, `dangerouslySetInnerHTML`, `document.write`, unescaped HTML templates `{!!`, `html_safe` |
| `open-redirect` | `Location:` / `redirect(` / `window.location` with request params |
| `authz` | `skip.*auth`, `permitAll`, `isAdmin`, session/jwt decode without verify, missing guard on route handlers |
| `jwt` | `jwt.encode`/`decode`, `jsonwebtoken`, `HS256` with hardcoded secret, `verify: false` |
| `cors` | `Access-Control-Allow-Origin` `*`, `cors({ origin: true })` |
| `webhook` | webhook/callback routes, missing signature verify (`timingSafeEqual`, HMAC) |
| `crypto` | `md5`, `sha1(`, `DES`, `ECB`, hardcoded IV, `Math.random` for tokens |
| `rce` | `eval(`, `Function(`, `pickle.loads`, `yaml.load(`, `deserialize`, `NSKeyedUnarchiver` |
| `ssrf-metadata` | `169.254.169.254`, `metadata.google`, `instance-data` |
| `unix-socket` | `unix://`, `control.sock`, `bind(` + `AF_UNIX` |
| `proto-rpc` | `grpc`, `ConnectRPC`, `.proto` service handlers |

Repo-specific extras (add hits, don't replace): Swift `NSTask`/`Process`,
`posix_spawn`; GitHub Actions `pull_request_target`; Terraform `0.0.0.0/0`.

## Scope files with no hits

`uncommitted` and `diff`: every scoped **source** file is still a
candidate (`vulnSlug: other-holistic`, empty `lineNumbers`). `full`:
matcher hits only, plus explicit entrypoints you listed in `INFO.md`
(HTTP routes, CLI verbs, socket servers) even if greps missed them.
