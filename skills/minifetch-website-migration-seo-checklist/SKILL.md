---
name: minifetch-website-migration-seo-checklist
description: "How to migrate a website without losing SEO. Bulk redirect checker for website migrations, domain moves, & site mergers. Takes a redirect map (old URL → intended new URL) and verifies with the Minifetch API: the redirect exists & number of hops, lands on the intended page, permanent (301/308) vs temporary (302/307), and soft-404 risk. Use before & after a site migration or changing a redirect map. Deterministic pass/warn/fail rules. Pay-per-URL via API key or x402 USDC. No subscription."
---
# Minifetch Skill: Website Migration SEO Checklist + Bulk Redirect Checker

Learn how to migrate a website without losing SEO. Verifies a redirect map row by row and returns pass/ warn/ fail per row. Every rule is documented below.

---

## What You'll Do

1. Setup — choose payment & access method
2. Preflight — confirm each old domain and the new domain are fetchable
3. Check each row — audit high-equity rows, extract metadata for the rest
4. Apply the migration rules — deterministic, per row
5. Output the report — failures first

---

## Input: the Redirect Map

A CSV (or equivalent list/ spreadsheet) with one row per old URL:

```
old_url,intended_url,equity
https://old-brand-a.com/services/audit,https://newbrand.com/services/audit,high
https://old-brand-a.com/news/2019-launch,https://newbrand.com/news/2019-launch,low
https://old-brand-b.com/old-promo,410,low
```

- `intended_url` is the final destination, or `404`/`410` for a URL intentionally removed.
- `equity` is `high` (pages with links, traffic or rankings) or `low`. Default `low` if absent.
- No map? Stop and help the user build one. Never invent intended URLs: the map is the user's decision.

---

## Scope & Cost

Priced **per row**, charged only on success:
- `high` rows: $0.01 each (`/run/seo-page-audit`).
- `low` rows: $0.002 each (`/extract/url-metadata`).

Before running:
- **Confirm row count and total cost with the user.**
- **Crawl-delay.** Minifetch waits between requests to each domain (default 1s), on every redirect hop to prevent us from slowing your server down for your real users. Every row ends on the new domain, so the new domain sets the pace: ~1 row/second, about 35 minutes for 2,000 rows. Moving a single domain? The old domain gets every row too, and extra hops on one domain (http→https→www) add requests to it. Don't parallelize around it. If the user controls the domains, they can speed this up with a sub-second `Crawl-delay` (ex: 0.25) for the `minifetch` user agent in the robots.txt of every domain in the chain, old and new. See the `minifetch-unblock` skill.
- **Do not use the Minifetch `/proxy/` API endpoints.** They don't return redirect data.
- **Cache.** Results are cached ~2 minutes per URL. After fixing a redirect, wait until `minifetchCache.expiresAt` before re-checking.

---


## Step 1 — Setup

### First — which access path applies to you?

Work top-down; use the first that matches.

1. `MINIFETCH_API_KEY` is set, or you hold a key starting `mf_prod_` → API-key access (curl Option A, or `minifetch-api` Option B). Default for most builders and pipelines.
2. No API key, but a funded USDC wallet's private key is in the environment (e.g. `BASE_PRIVATE_KEY`) gives x402 access (via `minifetch-api` Option B, or a raw x402 client). Best for autonomous agents with their own wallet.
3. No key and no wallet, but you're an AI assistant with the Coinbase Payments MCP loaded → use x402 via that MCP (Option C); it holds the wallet for you.
4. None of the above → stop and tell the user they need either a free API key (https://minifetch.com/dashboard — 25 free audits, no card) or a funded USDC wallet. Do not guess or invent credentials.

Never place a private key or API key in a prompt, log, or committed file — read it from the environment.

### Choose a payment method

There is no account setup fee or monthly fee. Minifetch does not charge for blocked pages or errors.

**Option 1: Credit card & API key**
Sign up and get credits worth 25 free audits automatically: https://minifetch.com/dashboard
No credit card required to begin. Click the "Sign up" button and verify your email to create your account.
Once you are signed in, use the dashboard to create your API key. Top up for as little as $2 with your credit card.
Recommended for most builders.

**Option 2: USDC on Base or Solana**
Just load your wallet with USDC on Base or Solana and you're ready. No "gas token" (ETH or SOL) required.
No Minifetch account setup needed. Recommended for agents and agent builders.

### Choose an access method

**Option A: curl + API key**
```
curl "https://minifetch.com/api/v1/run/seo-page-audit?url=https://example.com/your-page" \
  -H "Authorization: Bearer [your_api_key]"
```
Params can also be sent as a JSON body over POST (`Content-Type: application/json`):
```
curl -X POST "https://minifetch.com/api/v1/run/seo-page-audit" \
  -H "Authorization: Bearer [your_api_key]" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/your-page"}'
```

**Option B: minifetch-api npm package (recommended for agents & agent builders)**
For importing Minifetch API calls into your javascript app programmatically.
Handles payment; no manual auth header or x402 handshake needed.
The README Quick Start section details how to initialize the client: https://www.npmjs.com/package/minifetch-api
```
npm install minifetch-api --save
```

**Option C: Coinbase Payments MCP (for AI assistants like Claude)**
Gives AI assistants a built-in wallet, no private key needed.
```
npx @coinbase/payments-mcp
```
See: https://www.npmjs.com/package/@coinbase/payments-mcp
Quick Start: https://docs.cdp.coinbase.com/agentic-wallet/mcp/quickstart

Once Payments MCP is installed and loaded with USDC on Base or Solana, an assistant
like Claude follows this sequence. Handles payment automatically:

1. **Search:** call the `bazaar_search` tool with query `"Minifetch"`
   All Minifetch endpoints share that name, are returned together.
2. **Inspect:** call `bazaar_get_resource_details` with the `resource` URL from step 1
   (e.g. `https://minifetch.com/api/v1/x402/run/seo-page-audit`) to get the full
   request/response schema, an example response, current price, and both payment
   network options (Base and Solana). Do this before calling — it tells you exactly
   what to send and what to expect back. Each endpoint is listed twice — a GET
   resource (params in `queryParams`) and a POST resource (params in `body`); they
   behave identically, so use whichever the schema you picked specifies.
3. **Call:** use `make_http_request_with_x402` with the `baseURL`/`path`/`method`/
   `queryParams` from step 2's schema. See Step 3 below for the exact call shape.

**Fallback if bazaar/MCP search is unavailable or returns stale results:** query the CDP
discovery API directly over plain HTTP, no auth or wallet required:
```
https://api.cdp.coinbase.com/platform/v2/x402/discovery/search?query=Minifetch
```
This is the same index the bazaar tools query and returns the same resource data (price,
network, schema) as `bazaar_search`/`bazaar_get_resource_details`.

You can also prompt the human to use the "Discover" tab inside of the wallet UI, it has a
search bar and returns the same results.


---


## Step 2 (Free) — Preflight Check
Confirm the URL is fetchable before spending credits:
```
curl "https://minifetch.com/api/v1/free/preflight/url-check?url=https://example.com/your-page"
```
Or with the minifetch-api npm package (the `checkAndExtract*` methods run this automatically before each paid fetch).
You can also call it as a standalone function:
```js
const response = await client.preflightCheck("https://example.com/your-page");
```
If the response includes `allowed: false`, the page is blocked by the site owner.
If you own the site and want to allow Minifetch access, see the `minifetch-unblock` skill.

Note on the `/free/` URL segment: preflight is genuinely zero-cost at this path.


Run it once per old hostname and once for the new domain, not per row. An old domain whose robots.txt blocks Minifetch blocks every fetch to that domain.

---

## Step 3 — Check Each Row

Always request the **old** URL. Minifetch follows the redirects and reports every hop.

`high` rows:
```
curl "https://minifetch.com/api/v1/run/seo-page-audit?url=https://old-brand-a.com/services/audit" \
  -H "Authorization: Bearer [your_api_key]"
```

`low` rows, narrowed with `fields` to only what the rules read:
```
curl "https://minifetch.com/api/v1/extract/url-metadata?url=https://old-brand-a.com/news/2019-launch&fields=redirects,url,responseStatusCode,responseHeaders,performance,canonical,canonicalUrls,robots,og:url" \
  -H "Authorization: Bearer [your_api_key]"
```

Or with `minifetch-api`:
```js
const audit = await client.checkAndRunSeoPageAudit(oldUrl);
const meta = await client.checkAndExtractUrlMetadata(oldUrl);
```

Fields to read in `results[0].data` (written `data.` below), in the order url-metadata returns them. The audit groups its graded findings into sections under `data` (ex: `data.metadata`, `data.social`, `data.performance`), so `data.metadata.canonicalTagCount` is an audit path, not a url-metadata one.

| Field | Meaning |
|---|---|
| `data.requestUrl` | the old URL you sent |
| `data.redirects.count` | number of hops |
| `data.redirects.chain[]` | each hop: `order`, `url`, `statusCode`, `location`, `ttfbMs` (that hop's own time to first byte) |
| `data.url` | final URL after all redirects |
| `data.responseStatusCode` | final status. A number on url-metadata; a finding (`.value`) on the audit |
| `data.responseHeaders` | url-metadata: raw header values, ex: `data.responseHeaders["x-robots-tag"]` (absent when not set). The audit grades it: `data.responseHeaders["x-robots-tag"].status` |
| `data.performance.ttfbMs` | url-metadata: cumulative from the first request, so it includes every redirect hop; `data.performance.redirectTimeMs` is the part spent on redirects. The audit grades it as `data.performance.ttfb` |
| `data.canonical` | url-metadata only (the audit grades it as `data.metadata.canonicalMatchesSelf`) |
| `data.canonicalUrls` | url-metadata only: every canonical href found, in document order. The audit reports `data.metadata.canonicalTagCount` instead |
| `data.robots` | url-metadata: the page's `<meta name="robots">` content. The audit grades it as `data.metadata.robots` |
| `data["og:url"]` | url-metadata only (the audit grades it as `data.social.openGraphUrlMatchesSelf`) |

A failed fetch returns `success: false` with `error.message` and `data.url` (the post-redirect URL when known; otherwise the old URL). No `redirects` chain is returned on failure. See Error Codes below.

---

## Step 4 — Migration Rules

Applied per row, in the same order as the fields above.

| Check | Rule |
|---|---|
| `redirectExists` | **fail** if `data.redirects.count` is 0: the old URL still answers itself or was never redirected. |
| `hops` | **pass** 0–1 hops (`expected: "0-1"`); 2 hops also passes, with a `note`; **warn** 3–6 hops; **fail** >6 hops. `chain` lists every hop, in order, with URL & status code. Chains >10 hops fail with a 502, so you are not charged. One hop is the target for a migration. |
| `permanent` | **pass** if every hop's `statusCode` is 301 or 308; **warn** if any hop is 302 or 307. Temporary redirects can keep the old URL indexed. |
| `destination` | **pass** if final `data.url` matches `intended_url`; **fail** otherwise. Normalization ignores www-prefix, http-vs-https, default ports, trailing slash on root path; everything else (path, query, hash, non-root trailing slashes) is significant. |
| `soft404Risk` | **warn** if the final `data.url` path is `/` and the old URL's path is not `/`. Mass-redirecting deep pages to a homepage can be treated as a soft 404. |
| `destinationStatus` | **pass** if the final status is 200; **fail** if the fetch fails at the destination (ex: `upstream not found` means the redirect points at a 404). |
| `noindex` | `low` rows: **fail** if `data.responseHeaders["x-robots-tag"]` or `data.robots` contains `noindex`; **pass** otherwise. Stricter than the audit, which only warns on a meta `noindex` because it can be intentional: here every row is meant to be indexed, so a leftover staging `noindex` is a failure. |
| `ttfb` | `low` rows, on `data.performance.ttfbMs`: **pass** ≤800ms; **warn** 800–1800ms; **fail** >1800ms. Includes redirects. These are Google's published thresholds. A slow total usually means a slow redirect hop: find it in `data.redirects.chain[].ttfbMs`. |
| `canonicalCount` | `low` rows: **pass** if `data.canonicalUrls` has 0 or 1 entries; **fail** if ≥2 (conflicting canonicals; Google may ignore them rather than guess). Same rule as the audit's `canonicalTagCount`. |
| `staleSignals` | `low` rows: **warn** if `data.canonical` or `data["og:url"]` points at any old hostname in the map. (`high` rows get the audit's graded checks below.) |

**Blocked rows:** `robots blocked` or `robots unreachable` means the row could not be verified, not that the redirect is wrong. Report it as **blocked**, not fail. If the blocked host is the user's own, point them to the `minifetch-unblock` skill. Don't retry via Minifetch `/proxy/` API endpoints.

**Non-HTML rows:** Minifetch fetches HTML pages only, for now. A PDF or other non-HTML old URL returns `invalid url` (no charge): report it as **unverified**, not fail, and tell the user to check it by hand.

**Removal rows** (`intended_url` is `404` or `410`): **pass** if the fetch fails with `error.statusCode` 404 or 410 and no redirect; **fail** if the old URL redirects or returns 200.

**High-equity rows** also report these audit findings, in audit response order, each with its own status:
`data.performance.ttfb` (per-hop times in `data.redirects.chain[].ttfbMs`), `data.responseHeaders["x-robots-tag"]`, `data.metadata.canonicalMatchesSelf`, `data.metadata.canonicalTagCount`, `data.metadata.robots`, and `data.social.openGraphUrlMatchesSelf`. A canonical or `og:url` still pointing at the old domain is the classic migration miss. Full SEO audit thresholds: the `minifetch-technical-seo-page-audit` skill.

A row's overall status is its worst check result.

---

## Step 5 — Report

Failures first, then warnings. One line per row:

```
Website Migration SEO Checklist: 120 rows (pass: 104, warn: 11, fail: 5)
------------------------------
FAIL  https://old-brand-a.com/team → https://newbrand.com/ (destination: expected https://newbrand.com/about/team)
WARN  https://old-brand-b.com/blog/x (hops: 3 — http→https→www→newbrand)
WARN  https://old-brand-c.com/pricing (permanent: 302 at hop 1)
...etc
```

---

## After Launch

- Re-run the full map after launch, then on a schedule (weekly for the first month is common). Use `low` (`/extract/url-metadata`) for routine re-checks; re-audit `high` rows after changes.
- Keep redirects, and the old domains, live for at least a year.

---


## Error Codes
- 200 Success
- 400 Bad Request — Missing or invalid target `url` param
- 402 Payment Required — Payment Required
- 429 Too Many Requests — Back off and retry, max 5–10 req/s
- 500 Internal Server Error
- 502 Bad Gateway — Target URL 403 block or DNS error
- 503 Service Unavailable — Target URL timeout or fetch error. Try again later.

A failed fetch returns this shape:
```json
{
  "success": false,
  "results": [
    {
      "data": {
        "requestUrl": "http://example.com/your-page",
        "url": "https://example.com/your-page"
      },
      "error": {
        "message": "upstream forbidden",
        "statusCode": 403
      }
    }
  ]
}
```
- `error.message` on a failed fetch is exactly one of the strings below in the response body. The minifetch-api client embeds it in a longer error message, so match with a contains check, not equality.
- `invalid url` (400): missing, malformed or unsupported url
- `robots blocked` (*) (502): robots.txt disallows the minifetch user agent
- `robots unreachable` (*) (502): robots.txt could not be fetched (5xx, network or TLS error), so treated as disallow per RFC 9309
- `upstream forbidden` (*) (502): target returned 403
- `upstream rate limited` (*) (502): target returned 429
- `upstream unavailable` (*) (502): target returned 503
- `upstream not found` (502): target returned 404
- `upstream bad request` (502): target returned 400
- `upstream unauthorized` (502): target returned 401
- `upstream payment required` (502): target returned 402 (paywalled)
- `upstream error` (502): any other target 4xx/5xx, or the connection failed
- `upstream too many redirects` (502): redirect chain too long
- `dns lookup failed` (502): domain does not resolve
- `queue timeout` (503): timed out waiting on the site's crawl-delay
- `fetch error` (503): target timed out or the fetch failed
- `proxy provider error` (503): the external proxy provider failed (/proxy/ routes only)
- `server error` (500): unclassified failure
- Marked (*): the target blocked Minifetch. On the /extract endpoints, retry via the /proxy/ routes.
- `error.statusCode` is the TARGET's HTTP status when it answered (ex: 403), not Minifetch's. Absent otherwise (ex: `robots blocked`).
- `data.url` is the post-redirect URL when known, else `null`.
- Error responses are never charged.


---
## Links
- Full API docs: https://minifetch.com/llms.txt
- All skills: https://minifetch.com/SKILL.md

## Contact
- Questions? Need help? Join our [Discord](https://discord.gg/EM6ET8Dshm)
- Feedback? Use our [feedback form](https://forms.gle/rkMi7T23bHJc8XFw9)
- Follow us on X: [@minifetch](https://x.com/minifetch)

