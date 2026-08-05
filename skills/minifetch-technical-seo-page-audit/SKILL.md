---
name: minifetch-technical-seo-page-audit
description: "Run a deterministic technical SEO audit on a single web page through the Minifetch API. Use when a user wants to check one URL's indexability, metadata, canonicals, redirects, response headers, meta tags, structured data, TTFB performance, hreflang, headings, links, images, Open Graph/Twitter tags, and get prioritized pass/warn/fail findings against documented thresholds — no black-box scoring. Pay-per-URL via API key or x402 USDC; no subscription."
---
# Minifetch Skill: Technical SEO Page Audit

Run a full technical SEO audit on a single web page in one API call.
Returns a structured report with pass/ warn/ fail findings for every signal.
The audit flags long redirect chains, performance lags, missing or conflicting
canonicals, headers, robots directives and more to ensure indexability.
No black-box scoring — every threshold is documented and applied deterministically.

This is a **composer endpoint** (`/run/seo-page-audit`). Under the hood it
composes our /preflight check, extracts /url-metadata + /url-links, and finally
assembles all of that data into an audit report according to deterministic rules.
You pay one price for the whole web page audit, or call the individual pieces for a
fraction of the full audit price. If the page is blocked or errors, the request
fails and you are not charged.

**Base URL:** https://minifetch.com

Every endpoint below is reachable two ways: the plain `/api/v1/...` path (Option A/B
below — API key) and the `/api/v1/x402/...` path (Option C — USDC wallet,
no account). Same endpoint, same response, two payment rails — pick one per request.

---

## What You'll Do

1. Setup — choose payment and access method
2. Preflight (optional) — confirm the URL is fetchable
3. Run the audit — single API call, structured response
4. Interpret findings — every finding has a `status` and an `expected` value
5. Output the report — summarize pass/ warn/ fail counts and act on failures

---

## Scope & Cost

This audit is priced **per URL** ($0.01 each, charged only on success). Before auditing more than a page or two:

- **Confirm scope with the user first.** "Audit my site" is not a URL. Ask whether they mean one page, a specific list, or a section — and confirm before spending. Don't crawl a whole sitemap on your own initiative.
- **One page usually answers the question.** For "is this page healthy?", a single audit is enough. Batch only when the user actually wants many pages.
- **Filter cheaply before paying.** Run the free `/preflight/url-check` (Step 2) across candidate URLs to drop blocked or unreachable ones before spending on audits.
- **It's slow by design.** Minifetch honors each site's crawl-delay (default 1s between requests to a domain), so ~10 URLs takes at least ~10 seconds. This protects the site owner — don't try to parallelize around it. **If you own the site being audited**, you can go faster: set a sub-second `Crawl-delay` (e.g. `Crawl-delay: 0.5`) in its robots.txt and Minifetch will honor it — see the `minifetch-unblock` skill.

---


## Step 1 — Setup

### First — which access path applies to you?

Work top-down; use the first that matches.

1. `MINIFETCH_API_KEY` is set, or you hold a key starting `mf_prod_` → API-key access (curl Option A, or `minifetch-api` Option B). Default for most builders and pipelines.
2. No API key, but a funded USDC wallet's private key is in the environment (e.g. `BASE_PRIVATE_KEY`) → x402 access (`minifetch-api` Option B, or a raw x402 client). Best for autonomous agents with their own wallet.
3. No key and no wallet, but you're an AI assistant with the Coinbase Payments MCP loaded → x402 via that MCP (Option C); it holds the wallet for you.
4. None of the above → stop and tell the user they need either a free API key (https://minifetch.com/dashboard — 25 free audits, no card) or a funded USDC wallet. Do not guess or invent credentials.

Never place a private key or API key in a prompt, log, or committed file — read it from the environment.

### Choose a payment method

There is no account setup fee or monthly fee. Minifetch does not charge for blocked pages or errors.

**Option 1: Credit card & API key**
Sign up and get credits worth 25 free audits automatically: https://minifetch.com/dashboard
No credit card required to begin. Click the "Sign up" button and verify your email address to create your account.
Once you are signed in, use the dashboard to create your API key. Each successful fetch will be deducted from your credit balance.
Top up for as little as $2 with your credit card.
Recommended for most builders.

**Option 2: USDC on Base or Solana**
Just load your wallet with USDC on Base or Solana and you're ready. No "gas token" (ETH or SOL) required.
No Minifetch account setup needed. Recommended for agents and agent builders.

### Endpoints & Prices (pay-as-you-go)

| Endpoint | Price | Best for |
|---|---|---|
| `/api/v1/free/preflight/url-check` | Free | Check robots.txt before fetching |
| `/api/v1/run/seo-page-audit` | $0.01 | Run SEO audit on a web page |
| `/api/v1/extract/url-metadata` | $0.002 | Full structured SEO metadata |
| `/api/v1/extract/url-links` | $0.002 | All links + links metadata |
| `/api/v1/extract/url-preview` | $0.002 | Check how page unfurls when shared on social platforms, chat apps, and AI channels |
| `/api/v1/extract/url-content` | $0.002 | Check how page converts to clean markdown without nav, ads, scripts |

All `/api/v1/` endpoints also accept `/api/v1/x402/` for USDC crypto wallet payments on Base or Solana.


### Choose an access method

**Option A: curl + API key**
```
curl "https://minifetch.com/api/v1/run/seo-page-audit?url=https://yoursite.com/your-page" \
  -H "Authorization: Bearer [your_api_key]"
```

**Option B: minifetch-api (recommended for agents & agent builders)**
For importing Minifetch API endpoint calls into your application programmatically.
Handles payment automatically — no manual auth header or x402 handshake needed.
The README Quick Start section details how to initialize the client: https://www.npmjs.com/package/minifetch-api
```
npm install minifetch-api --save
```

**Option C: Coinbase Payments MCP (for AI assistants like Claude)**
Gives AI assistants a built-in wallet — no private key needed.
```
npx @coinbase/payments-mcp
```
See: https://www.npmjs.com/package/@coinbase/payments-mcp
Quick Start: https://docs.cdp.coinbase.com/agentic-wallet/mcp/quickstart

Once the Payments MCP is installed and loaded with USDC on Base or Solana, an assistant
like Claude follows this sequence — payment is handled automatically, no manual auth
header or x402 handshake needed:

1. **Search:** call the `bazaar_search` tool with query `"Minifetch"` -- all Minifetch endpoints share that name and are returned together.
2. **Inspect:** call `bazaar_get_resource_details` with the `resource` URL from step 1
   (e.g. `https://minifetch.com/api/v1/x402/run/seo-page-audit`) to get the full
   request/response schema, an example response, current price, and both payment
   network options (Base and Solana). Do this before calling — it tells you exactly
   what to send and what to expect back.
3. **Call:** use `make_http_request_with_x402` with the `baseURL`/`path`/`method`/
   `queryParams` from step 2's schema. See Step 3 below for the exact call shape for
   this audit endpoint.

**Fallback if bazaar/MCP search is unavailable or returns stale results:** query the CDP
discovery API directly over plain HTTP, no auth or wallet required:
```
https://api.cdp.coinbase.com/platform/v2/x402/discovery/search?query=Minifetch
```
This is the same underlying index the bazaar tools query and returns the same resource
data (price, network, schema) as `bazaar_search`/`bazaar_get_resource_details`.

You can also prompt the human to use the "Discover" tab inside of the wallet UI,
it has a search bar and returns the same results.


---


## Step 2 (Free) -- Preflight Check
Confirm the URL is fetchable before spending credits:
```
curl "https://minifetch.com/api/v1/free/preflight/url-check?url=https://yoursite.com/your-page"
```
Or with minifetch-api (the `checkAndExtract*` methods run this automatically before each paid fetch).
You can also call it as a standalone function:
```js
const response = await client.preflightCheck("https://yoursite.com/your-page");
```
If the response includes `allowed: false`, the page is blocked by the site owner.
If you own the site and want to allow Minifetch access, see the `minifetch-unblock` skill.

Note on the `/free/` URL segment: preflight is genuinely zero-cost at this path, and
this is the one to use in normal workflows. A second, nominally-priced ($0.001) version
of the same check is separately registered at `/api/v1/x402/preflight/url-check` — the
CDP Bazaar only lists paid x402 resources, so a $0 endpoint can't be discovered through
bazaar/MCP search at all. The paid version exists purely so agents doing bazaar-driven
discovery can find Minifetch's preflight check exists; call the free `/free/` path
directly once you know about it.


The audit endpoint runs preflight internally and returns 502 (no charge) if
the URL is blocked, so calling `/url-check` first is optional. It can still
be useful when you have a list of candidate URLs and want to filter cheaply
before paying.

---

## Step 3 — Run the Audit

**Price:** $0.01 per URL (charged only on success).

From your CLI:
```
curl "https://minifetch.com/api/v1/run/seo-page-audit?url=https://yoursite.com/your-page" \
  -H "Authorization: Bearer [your_api_key]"
```

Or with `minifetch-api`: https://www.npmjs.com/package/minifetch-api
```
const response = await client.checkAndRunSeoPageAudit("https://yoursite.com/your-page");
```

Or via Coinbase Payments MCP (Option C above) — after `bazaar_search` and
`bazaar_get_resource_details` as described in Step 1, call `make_http_request_with_x402`:
```
baseURL: "https://minifetch.com"
method: "GET"
path: "/api/v1/x402/run/seo-page-audit"
queryParams: { "url": "https://yoursite.com/your-page" }
```
The MCP signs and settles the USDC payment itself — no separate auth step.

Example response (a real audit — long link arrays trimmed):
```json
{
  "success": true,
  "results": [
    {
      "data": {
        "summary": { "pass": 20, "warn": 5, "fail": 1 },
        "requestUrl": "https://anthropic.com",
        "url": "https://www.anthropic.com/",
        "responseStatusCode": { "status": "pass", "expected": 200, "value": 200 },
        "redirects": {
          "status": "pass",
          "expected": "0-1",
          "count": 1,
          "chain": [{ "order": 1, "url": "https://anthropic.com", "statusCode": 301 }]
        },
        "performance": {
          "ttfb": { "status": "pass", "expected": "≤800ms", "redirectTimeMs": 139, "ttfbMs": 250 },
          "responseTimeMs": 260
        },
        "responseHeaders": {
          "date": { "value": "Tue, 14 Jul 2026 23:05:42 GMT" },
          "last-modified": { "value": "Tue, 14 Jul 2026 15:41:25 GMT" },
          "link": { "value": "<https://cdn.prod.website-files.com>; rel=preconnect; crossorigin" },
          "x-robots-tag": { "status": "pass", "expected": "no noindex directive", "value": null },
          "content-type": { "status": "pass", "expected": "text/html", "value": "text/html; charset=utf-8" },
          "cache-control": { "status": "warn", "expected": "present", "value": null },
          "strict-transport-security": { "status": "pass", "expected": "present", "present": true }
        },
        "compliance": {
          "robotsTxt": { "status": "pass", "expected": "allowed", "allowed": true, "userAgent": "minifetch/1.0 (+https://minifetch.com/site-owner-faq)" },
          "https": { "status": "pass", "expected": "https", "value": "https" },
          "mixedContent": {
            "status": "fail",
            "expected": 0,
            "count": 3,
            "resources": ["http://trust.anthropic.com/", "http://trust.anthropic.com/", "http://claude.com/platform/api"],
            "omitted": 0
          }
        },
        "metadata": {
          "title": { "status": "warn", "expected": "30-60", "value": "Home | Anthropic", "length": 16 },
          "description": { "status": "pass", "expected": "70-155", "value": "Anthropic is an AI safety and research company that's working to build reliable, interpretable, and steerable AI systems.", "length": 121 },
          "canonical": { "status": "pass", "expected": "present", "value": "https://www.anthropic.com", "source": "html", "malformed": false, "conflictWithLinkHeader": false },
          "canonicalMatchesSelf": { "status": "pass", "expected": true, "value": true, "canonicalUrl": "https://www.anthropic.com/", "crossDomain": false },
          "canonicalTagCount": { "status": "pass", "expected": "≤1", "count": 1, "canonicalUrls": ["https://www.anthropic.com"] },
          "robots": { "status": "pass", "expected": "no noindex directive", "value": "index, follow" },
          "lang": { "status": "pass", "expected": "present", "value": "en" },
          "viewport": { "status": "pass", "expected": "present", "value": "width=device-width, initial-scale=1" }
        },
        "hreflang": { "note": "no hreflang tags declared", "count": 0 },
        "jsonld": { "status": "warn", "expected": "≥1 item", "itemCount": 0, "types": [], "nestedTypes": [] },
        "headings": {
          "h1": { "status": "pass", "expected": 1, "count": 1 },
          "hierarchy": { "status": "warn", "expected": "no skips", "skips": ["h3-h6"] }
        },
        "content": { "wordCount": 702, "contentHtmlRatio": 2.4 },
        "images": {
          "total": 0,
          "missingAlt": { "status": "pass", "expected": 0, "count": 0 },
          "missingDimensions": { "status": "pass", "expected": 0, "count": 0 }
        },
        "links": {
          "total": 176,
          "anchorCount": 4,
          "nofollowCount": 0,
          "internal": {
            "count": 79,
            "uniqueTargets": 10,
            "topInternalTargets": [
              { "url": "https://www.anthropic.com/research", "count": 4, "anchorTexts": [{ "text": "Research", "count": 3 }, { "text": "research", "count": 1 }] },
              { "url": "https://www.anthropic.com/constitution", "count": 4, "anchorTexts": [{ "text": "Claude's Constitution", "count": 3 }] }
            ]
          },
          "external": {
            "count": 97,
            "uniqueDomains": 9,
            "topExternalDomains": [{ "domain": "claude.com", "count": 71 }, { "domain": "claude.ai", "count": 9 }]
          },
          "emptyLinkText": { "status": "warn", "expected": 0, "count": 7 }
        },
        "social": {
          "openGraph": { "status": "pass", "expected": "og:title, og:description, og:image, og:type", "present": ["og:title", "og:description", "og:image", "og:type"], "missing": [] },
          "openGraphUrlMatchesSelf": { "note": "no og:url declared" },
          "twitterCard": { "status": "pass", "expected": "twitter:card, twitter:title, twitter:image", "present": ["twitter:card", "twitter:title"], "presentViaOpenGraphFallback": ["twitter:image"], "missing": [] }
        },
        "minifetchCache": { "hit": false, "cachedAt": "2026-07-14T23:05:45.547Z", "expiresAt": "2026-07-14T23:07:45.547Z" }
      }
    }
  ]
}
```
The response above is a real audit (Anthropic's home page), abbreviated only by trimming the `topInternalTargets`/`topExternalDomains` arrays to two entries each — they hold up to 10 (see Step 4). It is valid, parseable JSON: no comments, no `...` elisions.

Every audit finding has the same shape:
```json
{
  "status": "pass" | "warn" | "fail",
  "expected": <expected value>,
  ... // additional fields (value, count, length, etc.)
}
```

Pure data fields (counts, dates, dimensions) appear without `status` or
`expected` — they are informational pass-throughs. Some findings drop to
informational (a `note`, no `status`) when there's nothing to evaluate:
`canonical`, `canonicalMatchesSelf`, `openGraphUrlMatchesSelf`, and `hreflang`
are always present in the response but carry a `note` instead of a `status`
when not applicable. Null-check `status` before relying on it.

---

## Step 4 — Audit Rules

Every threshold is documented here. The audit utility applies these
deterministically. No scoring model, no AI judgment.

The audit composes /preflight/url-check + /extract/url-metadata +
/extract/url-links data + the rules below. While iterating, call the primitives
directly — they're cheaper and return the same underlying data. Watch the
`minifetchCache.hit` field — back-to-back calls within the cache window
(~2 min) skip the network fetch entirely. Run the full audit composer
again once your pipeline is stable and the cache `expiresAt` timestamp
has passed. All API endpoints share the cache.

### summary
`{ pass, warn, fail }` — counts of findings with each status across the whole report. Pure data fields (counts, dates) are not counted.

### responseStatusCode
**pass** when 200; **fail** otherwise. (3xx redirects are followed before this check.)

*Why this matters:*
- The page should return HTTP 200. Redirects (3xx) are followed before this check, so anything other than 200 here is a real error.

### redirects
**pass** 0–2 hops (2 comes back with a `note`, still counted as a pass but flagged); **warn** 3–6 hops; **fail** above 6 hops. `chain` lists every hop followed, in order, with the URL and status code of each. Chains longer than the configured maximum (10 hops by default) are not reachable here and the request fails with a 502 (`"upstream too many redirects"`) before an audit is produced, so you are not charged for it.

*Why this matters:*
- How many redirect hops were followed to reach the final page, and the chain itself (each hop's URL and status code). 0-1 hops is normal (a single https or www. redirect not a problem). 2 hops is still acceptable, but worth a look. 3 or more hops wastes search engine crawl budget, diluting link equity passed through the chain; Google has said it will only follow up to 5 redirect hops per crawl attempt, so long chains risk not being followed to the destination at all.

### performance
One graded finding (`ttfb`) plus one informational field (`responseTimeMs`).

| Field | Rule |
|---|---|
| `ttfb.redirectTimeMs` | informational — the portion of `ttfbMs` spent on redirect hops before the final one began. Always present; `0` when there were no redirects. |
| `ttfb.ttfbMs` | **pass** ≤800ms; **warn** 800–1800ms; **fail** >1800ms. This is Google's own published Good/Needs Improvement/Poor thresholds. |
| `responseTimeMs` | informational — `ttfb` plus full body download time. Not graded: no equivalently authoritative single threshold the way TTFB has one. |

*Why this matters:*
- **ttfb** — Time to First Byte, Google's own published metric (web.dev): the time between navigation start and the first byte of the response arriving. Good is 800ms or less, needs improvement is 800–1800ms, and poor is over 1800ms. Measured from our server in North America.
- **responseTimeMs** — Total time from the start of the final request (after redirects) to the full response body being read — ttfbMs plus body download time. Reported for context but not graded: unlike TTFB, there is no single authoritative threshold for full response time the way Google publishes one for TTFB.

### responseHeaders
| Header | Rule |
|---|---|
| `Date`, `Last-Modified` | informational — raw response header values, no status |
| `Link` | informational only — no status |
| `X-Robots-Tag` | **fail** if contains `noindex`; **pass** otherwise |
| `Content-Type` | **pass** if matches `text/html`; **fail** otherwise |
| `Cache-Control` | **pass** if present; **warn** if missing |
| `Strict-Transport-Security` | **pass** if present (HSTS); **warn** if missing |

*Why these matter:*
- **Link** — The Link response header can declare a canonical URL outside the HTML using rel="canonical", e.g. Link: <https://example.com/page>; rel="canonical". Servers sometimes use this instead of, or alongside, an HTML <link rel="canonical"> tag. When both are present they must agree — see the canonical finding, which reconciles the two and fails on a conflict.
- **X-Robots-Tag** — A server-level robots directive sent as an HTTP header. If it contains noindex, search engines will not index the page even if the HTML and robots meta tag say otherwise.
- **Content-Type** — An SEO audit target should be served as text/html. PDFs, JSON, or other content types are not crawled the same way.
- **Cache-Control** — Tells browsers and intermediaries how long to cache the response. Missing cache-control means clients fall back to heuristic defaults, which can hurt repeat-visit performance.
- **Strict-Transport-Security** — HSTS header tells browsers to only connect to this domain over HTTPS. A security best practice and a minor ranking signal.

### compliance
| Field | Rule |
|---|---|
| `robotsTxt` | **pass** if site's robots.txt allows our user agent (`minifetch`); **fail** if disallowed (request returns 502, no charge) |
| `https` | **pass** if the audited page is served over HTTPS (checked against the post-redirect URL); **fail** otherwise. `value` is one of `https`, `http`, or `unknown`. |
| `mixedContent` | **pass** if 0 http:// resources; **fail** if any. Scans the HTML for `src`/`href`/`data` attributes with http:// values. `resources` is the first 20 offending URLs; `omitted` is any beyond. On non-HTTPS pages this finding is a no-op pass with a `note` field — the `https` finding above is the real issue there. |

*Why this matters:*
- **robotsTxt** — The site's robots.txt must allow our user agent. If disallowed, the audit returns 502 and is not charged.
- **https** — SEO-relevant pages should be served over HTTPS. Search engines penalize http pages, browsers show 'Not Secure' warnings, and mixed content protections only activate over HTTPS. This finding looks at the URL we actually fetched (post-redirect), so an audit of http://example.com that redirects to https://example.com passes.
- **mixedContent** — On an HTTPS page, any resource loaded over http:// is mixed content. Browsers block active mixed content (scripts, iframes, stylesheets) and warn on passive (images, video). The resources array lists the first 20 offending URLs so you know what to fix; the count and omitted fields tell you the full total.

### metadata
| Field | Rule |
|---|---|
| `title` | **pass** 30–60 chars; **warn** 1–29 (short — room for keywords) or 61–70 (risks truncation); **fail** empty or >70. |
| `description` | **pass** 70–155 chars; **warn** 1–69 (short — room for USPs/CTA) or 156–200 (risks truncation); **fail** empty or >200. |
| `canonical` | **pass** if present, parseable, and consistent; **fail** if the HTML and Link response header values disagree (`conflictWithLinkHeader: true`) or the canonical is unparseable (`malformed: true`); **info** (no status) if absent — search engines self-canonicalize a page to its own URL, so a missing canonical only matters when the page has duplicate URLs the audit can't see. Source is one of `html`, `header`, `both`. |
| `canonicalMatchesSelf` | **pass** if the canonical resolves to the *post-redirect* final URL we fetched (`value: true`); **warn** if it points elsewhere (`value: false`); informational (a `note`, no status) when the canonical is absent or unparseable — the `canonical` finding above carries that detail. Pointing elsewhere is often intentional for paginated, filtered, or syndicated pages — a warn, not a fail. The `crossDomain` boolean is informational (does the canonical point off-domain?). Normalization for the comparison ignores www-prefix, http-vs-https, default ports, and trailing slash on root path; everything else (path, query, hash, non-root trailing slashes) is significant. Relative canonicals are resolved against the fetched URL like a browser would. |
| `canonicalTagCount` | A separate question from `canonical` above: not "is there one" but "are there too many." **pass** if 0 or 1 `<link rel="canonical">` tags are found on the page; **fail** if 2 or more, regardless of whether they agree on the URL — this is the case where Google may ignore canonicalization on the page altogether rather than guess which one you meant. `canonicalUrls` always lists every raw href found, in document order, whatever the count. |
| `robots` | **warn** if value contains `noindex`; **pass** otherwise. Defaults to `"index, follow"` (Google's assumed default) when meta tag is absent. Same reasoning as `canonicalMatchesSelf`: `noindex` is often intentional (admin pages, staging, internal search results) so we surface it as a warn, not a fail. |
| `lang` | **pass** if present; **warn** if missing. Attribute on top-level `<html>` tag. |
| `viewport` | **pass** if present; **warn** if missing |

*Why these matter:*
- **title** — The clickable headline shown in Google search results. 30–60 characters is the healthy range, and the upper end (50–60) uses the available width best. Over ~60 characters Google truncates the title and may drop important words. A short title is not broken — it renders fine — but it leaves room to add keywords or communicate the page value, so we flag it as an opportunity rather than an error.
- **description** — Often used as the snippet under the title in search results. 70–155 characters is the healthy range. Over ~155 characters Google truncates the snippet; under 70 it still renders fine but leaves room to add benefits, USPs, or a call to action — so a short description is flagged as a missed opportunity, not an error. Google may rewrite the description regardless, but a strong one still influences click-through.
- **canonical** — Tells search engines which URL is the master version when duplicate content exists across multiple URLs. Can appear in the HTML or the Link response header. If both are present they must agree (a disagreement is a fail), and the canonical must be a parseable URL (an unparseable one is a fail, flagged by malformed: true). If no canonical is declared, search engines self-canonicalize the page to its own URL, which is fine unless the page has duplicate URLs the audit cannot see. So a missing canonical is surfaced as information, not graded.
- **canonicalMatchesSelf** — Whether the declared canonical points back at the URL we fetched: value true means it points at this page, false means it points elsewhere. When the canonical is missing or unparseable there's nothing to compare, so this is informational (a note, no status); the canonical finding above carries the detail. If it points elsewhere (false), Google likely treats this page as a duplicate, consolidates ranking signals to the canonical target, and usually shows that URL instead — though a cross-domain canonical is a strong hint Google can still override. Pointing elsewhere is usually deliberate (syndicated content crediting the original), so it's a warn, not a fail. The crossDomain field flags an off-domain canonical as informational, doesn't change status.
- **canonicalTagCount** — How many <link rel="canonical"> tags are on the page, separately from whether the declared canonical (above) is valid. 0 or 1 is a pass. More than one is a fail regardless of whether the extra tags agree on the URL — search engines may ignore canonicalization on the page altogether rather than guess which one you meant. canonicalUrls lists every raw href found, in document order.
- **robots** — The robots meta tag controls whether search engines index this page and follow its links. If absent, Google assumes "index, follow". A noindex value keeps the page out of search results; often intentional (admin pages, staging environments, internal search results) which is why this surfaces as a warn rather than a fail. Same logic as canonicalMatchesSelf: page is opting out of indexing, we surface it but don't presume to grade it.
- **lang** — The lang attribute on the top-level `<html>` tag tells search engines and screen readers what language the page is in. Helps with accessibility and international SEO.
- **viewport** — The viewport meta tag tells mobile browsers how to size the page. Pages without it are flagged as not mobile-friendly, which is a ranking factor.

### hreflang (always present; informational `note` + `count: 0` when the page has no hreflang tags)
| Field | Rule |
|---|---|
| `count` | informational — number of hreflang entries on the page |
| `xDefault` | informational — `present: true/false` |
| `selfReferencing` | **pass** if at least one entry's href matches the audited URL; **fail** otherwise. Matched against the *post-redirect* final URL, not the URL you requested (if there were redirects) |
| `fullyQualifiedUrls` | **pass** if all hrefs are absolute (`http://` or `https://`); **fail** with the offending hrefs in `invalid` |
| `inHead` | **pass** if all hreflang `<link>` tags appear inside `<head>`; **warn** otherwise |

*Why these matter:*
- **selfReferencing** — A page's hreflang set must include an entry that points back to itself. Without self-reference, Google treats the entire set as invalid.
- **fullyQualifiedUrls** — hreflang hrefs must be absolute URLs (with http:// or https://). Relative URLs are silently ignored by search engines.
- **inHead** — hreflang `<link>` tags must appear inside `<head>`. Tags placed in `<body>` are ignored.

### jsonld
**pass** if at least one typed item is present; **warn** if none. `itemCount`
is the number of top-level items Google evaluates — `@graph` arrays are
expanded so each node counts as its own item. `types` lists the distinct
top-level item types; `nestedTypes` lists supporting entity types found inside
those items (an author Person, a logo ImageObject, breadcrumb ListItems) and is
informational. `itemCount` and `types.length` can differ (two Product items =
itemCount 2, one type).

*Why this matters:*
- JSON-LD structured data makes pages eligible for rich results in search (recipe cards, product listings, review stars, etc). itemCount is the number of top-level items Google can evaluate. @graph arrays are expanded so each node counts as its own item, the way Google reads them. types lists the distinct top-level item types; nestedTypes lists supporting entity types found inside those items (an author Person, a logo ImageObject, breadcrumb ListItems) and is informational, since Google treats them as properties of their parent item rather than standalone items. At least one typed item makes the page eligible.

### headings
| Field | Rule |
|---|---|
| `h1` | **pass** if exactly 1; **fail** otherwise |
| `hierarchy` | **pass** if no level skips in document order (h2 → h4 is a skip); **warn** if any skips found, listed in `skips` |

*Why these matter:*
- **h1** — Search engines treat the h1 as the page's primary topic. Multiple h1s dilute the signal; zero h1s leave the page without a clear subject.
- **hierarchy** — Heading levels should descend without skipping (h2 → h3, not h2 → h4). Skipped levels confuse screen readers and weaken the document's logical structure.

### content
Purely informational — no pass/warn/fail status. Both fields are derived from the response body with no extra network fetch.

| Field | Description |
|---|---|
| `wordCount` | Visible text word count after stripping HTML tags, `<script>` blocks, `<style>` blocks, and HTML entities. Under ~300 words is the classic thin-content threshold, but context matters: a 150-word product page is not thin content. |
| `contentHtmlRatio` | Visible text bytes ÷ total HTML bytes, as a percentage (0–100, 1 decimal). A ratio under ~10% suggests the page is mostly markup and boilerplate. Complements `wordCount`. |

### images (uses `imgTags` from url-metadata)
| Field | Rule |
|---|---|
| `missingAlt` | **pass** if 0; **warn** if any. Empty `alt=""` counts as missing — decorative images legitimately use empty alt, but on SEO-targeted pages this is rare and worth flagging. |
| `missingDimensions` | **pass** if 0; **warn** if any. Width+height attributes prevent CLS (Core Web Vitals). |

*Why these matter:*
- **missingAlt** — Alt text describes images for screen readers and shows up when images fail to load. Search engines also use it to understand image content. Empty alt="" is counted as missing here — decorative images legitimately use empty alt, but on SEO-targeted pages this is rare and worth flagging.
- **missingDimensions** — Width and height attributes prevent Cumulative Layout Shift (CLS), a Core Web Vitals metric and a ranking factor. Without them, the page jumps around as images load.

### links (uses /extract/url-links endpoint raw data)
`internal.count`, `external.count`, and `total` are passthrough — no status. Top-level `anchorCount` (in-page `#fragment` links) and `nofollowCount` are also informational.

The `internal` and `external` objects include richer detail beyond raw counts: `internal.topInternalTargets` lists the top 10 most-linked-to internal URLs from this page along with the anchor texts used for each (deduped by origin+pathname, so query strings and fragments don't fragment the count).
`external.topExternalDomains` lists the top 10 external domains and how many times each was linked. Useful for spotting over-linking and anchor-text-diversity issues.

| Field | Rule |
|---|---|
| `anchorCount` | informational — number of in-page `#fragment` links |
| `nofollowCount` | informational — number of links with `rel="nofollow"` |
| `internal.topInternalTargets` | informational — top 10 internal URLs by link frequency, with anchor text variants |
| `external.topExternalDomains` | informational — top 10 external domains by link frequency |
| `emptyLinkText` | **pass** if 0; **warn** if any. Counts links with no visible text, no wrapped image, and no `aria-label`. Icon/SVG links with `aria-label` are accessible and not counted. |

*Why this matters:*
- **emptyLinkText** — Counts links with no visible text, no wrapped image, and no aria-label attribute. Screen readers and search engines have nothing to announce or index for these. Icon and SVG links with aria-label are accessible and excluded from this count. Empty link text is also a Google ranking concern because Google uses anchor text to understand what a link points to.

### social
Required Open Graph: `og:title`, `og:description`, `og:image`, `og:type`.
Twitter Card (ideal, but each falls back to Open Graph): `twitter:card`, `twitter:title`, `twitter:image`.

| Section | Rule |
|---|---|
| `openGraph` | **pass** if all 4 present; **warn** if 1–2 missing; **fail** if 3+ missing |
| `openGraphUrlMatchesSelf` | **pass** if `og:url` resolves to the page we fetched (`value: true`); **warn** if it points elsewhere (`value: false`); **fail** if `og:url` is present but unparseable (surfaced as a `note`, no value). Informational (a `note`, no status) when `og:url` is absent — social platforms fall back to the shared URL, so that's a non-issue. Same normalization rules and `crossDomain` informational field as `canonicalMatchesSelf`. |
| `twitterCard` | **pass** if all 3 fields are covered; **fail** if all 3 missing; **warn** if 1–2 missing. A field is *covered* when the twitter tag is present **or** its Open Graph fallback is — `twitter:title`→`og:title`, `twitter:image`→`og:image`, `twitter:card`→`og:image`. So a page with good Open Graph tags passes even with no twitter-specific tags. `presentViaOpenGraphFallback` lists fields covered by OG rather than declared directly. |

*Why these matter:*
- **openGraph** — Open Graph tags control how the page renders when shared on Facebook, LinkedIn, Slack, and most other platforms. og:title, og:description, og:image, and og:type are the minimum required set.
- **openGraphUrlMatchesSelf** — Whether og:url points back at the URL we fetched: value true means it points at this page, false means elsewhere. Social platforms use og:url as the 'canonical URL for sharing'; it determines which URL accumulates engagement (likes, shares) and is shown as the destination in the share card. Pointing elsewhere (false) is a warn, often intentional for syndicated content. When og:url is absent this is informational (a note). Platforms fall back to the shared URL, a non-issue. A present-but-unparseable og:url is a fail (a note, no boolean value).
- **twitterCard** — Twitter Card tags control rendering on X / Twitter. twitter:card, twitter:title, and twitter:image are the ideal set, but X falls back to Open Graph when a twitter tag is absent — twitter:title to og:title, twitter:image to og:image, and the card itself renders off og:image. This audit credits that fallback: a twitter tag is only counted as missing when its Open Graph equivalent is also absent, so a page with solid Open Graph tags passes even with no twitter-specific tags. The presentViaOpenGraphFallback field lists which fields are covered by Open Graph rather than declared directly.

---

## Step 5 — Working with Audit Results

The audit response is a structured JSON document with `pass`/`warn`/`fail`
findings throughout. Common patterns:

**Triage by status.** Filter for `fail` first, then `warn`. Pure data
fields without `status` are informational.

**Rendering for humans.** If you're presenting the audit to an end user
rather than feeding it back into a pipeline, a flat one-line-per-finding
summary reads better than the raw JSON. For example:

```
SEO Audit: https://yoursite.com/your-page (pass: 6, warn: 4, fail: 1)
------------------------------
PASS  compliance.robotsTxt
FAIL  compliance.mixedContent (found 2, expected 0)
PASS  responseStatusCode (200)
PASS  metadata.title (54 chars, expected 30-60)
WARN  metadata.description (60 chars, expected 70-155)
PASS  metadata.canonical (html, expected present)
PASS  headings.h1 (1 found, expected 1)
WARN  headings.hierarchy.skips (h1-h3)
WARN  social.openGraph (missing: og:image)
PASS  jsonld (2 items: Article, BreadcrumbList)
WARN  images.missingAlt (3 images)
..etc
```

---

## Iterating on Results

**Compose with other endpoints.** Use `/extract/url-metadata`  or
`/extract/url-links` (full link list) when an audit finding needs deeper
inspection. While iterating or monitoring, you can call the primitives directly;
they cost a fraction of the full audit and return much of the same underlying data.
The API primitives are also designed to be cheap and token-efficient, offering cost
savings on both ends: lower Minifetch price as well as AI compute cost. Fetch only
what you need, when you need it.

Details on the API primitives are in the full API docs: https://minifetch.com/llms.txt

Every audit response includes a `minifetchCache` object:
```json
"minifetchCache": {
  "hit": false,
  "cachedAt": "2026-02-18T22:37:32.889Z",
  "expiresAt": "2026-02-18T22:39:32.889Z"
}
```

- `hit: true` means the underlying page was served from cache — the
  page was not re-fetched.
- The cache window is typically 2 minutes. Use `expiresAt` as your
  retry-after value, not a fixed delay.
- All Minifetch API endpoints share the cache, keyed by URL.

---


## Error Codes
- 200 Success
- 400 Bad Request — Missing or invalid target `url` param
- 402 Payment Required — Payment Required
- 429 Too Many Requests — Back off and retry, max 5–10 req/s
- 500 Internal Server Error
- 502 Bad Gateway — Target URL 403 block or DNS error
- 503 Service Unavailable — Target URL timeout or fetch error. Try again later.

All non-2xx responses share this shape:
```json
{
  "success": false,
  "results": [
    {
      "data": { "requestUrl": "http://yoursite.com/your-page", "url": "https://www.yoursite.com/your-page" },
      "error": {
        "message": "robots blocked",
        "statusCode": 502
      }
    }
  ]
}
```
`error.message` is a short string (e.g. `"robots blocked"`, `"invalid url"`,
`"upstream too many redirects"`, `"dns lookup failed"`) — safe to match on programmatically.
`error.statusCode` is always present on error.
`data.url` is the post-redirect URL when known, else `null`; you are not charged for
non-2xx responses.



---

## Links
- Full API docs: https://minifetch.com/llms.txt
- All skills: https://minifetch.com/SKILL.md

## Contact
- Questions or need help? Join our [Discord server](https://discord.gg/EM6ET8Dshm).
- Feedback or bulk credits waitlist? Use our [feedback form](https://forms.gle/rkMi7T23bHJc8XFw9).
- Follow us on X: [@minifetch](https://x.com/minifetch)


