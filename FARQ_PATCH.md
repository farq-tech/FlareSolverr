# Farq JSON POST patch (P0-HS-AUTH)

Based on upstream FlareSolverr `master` + ideas from [PR #1542](https://github.com/FlareSolverr/FlareSolverr/pull/1542) (ONEX1/`custom-json-support`).

**Do not deploy raw #1542** — it drops turnstile/`disableMedia`, rebases many unrelated files, and overwrites XHR `responseText` with `page_source`.

## Behavior

- Re-enable `headers` as a **dict** on `request.*`
- `request.post` with `Content-Type: application/json` (header or `contentType`):
  1. Navigate to site origin (skipped if session already same-origin)
  2. Solve CF as usual
  3. Fire in-browser XHR POST with custom headers + JSON body
  4. Return XHR **status + responseText** in `solution`
- Form-urlencoded `request.post` unchanged (HTML form hack)
- Turnstile / disableMedia / sessions preserved

## Durable deploy source

- GitHub: https://github.com/farq-tech/FlareSolverr (fork of FlareSolverr/FlareSolverr)
- Branch: `farq-json-post`
- Version string: `3.5.0+farq-json-post`
- Railway project **Farq** (`b260dd2a-6eb9-416c-a717-80a652cdb199`) / service **flaresolverr**
- Build: Dockerfile from GitHub branch (not ephemeral `railway up`)
- Private URL: `http://flaresolverr.railway.internal:8191`

Dockerfile has no `VOLUME` line (Railway rejects it); `/config` is created in-image and writable.

## Provenance

Patch copies / patches also live in the Farq monorepo under `ops/flaresolverr-json-post/` for audit.
