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

## Deployed

- Railway project **Farq** / service **flaresolverr**
- Version string: `3.5.0+farq-json-post`
- Deploy method: `railway up` from patched tree (Dockerfile `VOLUME` removed — Railway rejects it)
- Private URL: `http://flaresolverr.railway.internal:8191`

### Rebuild from this folder

```bash
# Apply patches onto a fresh FlareSolverr clone, or use the .py copies here as the
# src/ replacements, then:
cd /path/to/patched-flaresolverr
# ensure Dockerfile has no VOLUME line
railway link -p Farq -e production -s flaresolverr
railway up -s flaresolverr -e production --ci
```

## HUMAN ACTION (durable image source)

Railway currently serves the CLI-uploaded build. For a durable GitHub-sourced image:

1. On GitHub: **Fork** `https://github.com/FlareSolverr/FlareSolverr` → `farq-tech/FlareSolverr` (or similar).
2. Push branch `farq-json-post` with this patch (`dtos.py`, `flaresolverr_service.py`, Dockerfile without `VOLUME`, version bump in `package.json`).
3. In Railway UI → service `flaresolverr` → Settings → connect that repo/branch (replace `ghcr.io/flaresolverr/flaresolverr:latest`).
4. Delete accidental empty Railway project created during deploy experiments named **patched** (id `6ccaec80-7475-47a6-b781-328d54e365a6`) if still present.

## Oregon probe (2026-08-30)

Session GET `https://hungerstation.com/` → `cf_clearance` present.  
Same-session JSON POST `/api/v3/verification/refresh` → **HTTP 401** `unauthorized` (API reached — **not CF**).  
Verdict: `AUTH_REJECTED_NOT_CF` (credential/header contract; do not rotate tokens until directed).
