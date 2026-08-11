---
name: minifetch-technical-seo-page-audit
description: "Run a deterministic technical SEO audit on a single web page through the Minifetch API. Use when a user wants to check one URL's indexability, canonicals, redirects, response headers, meta tags, structured data, TTFB performance, hreflang, headings, links, images, Open Graph/Twitter tags, and get prioritized pass/warn/fail findings against documented thresholds. No black-box scoring. Pay-per-URL via API key or x402 USDC. No subscription."
---
# Minifetch Skill: Technical SEO Page Audit

Returns a structured JSON report with pass/ warn/ fail findings for every signal. Thresholds documented below.

---

## What You'll Do

1. Setup — choose payment & access method
2. Preflight (optional) — confirm URL is fetchable
3. Run the audit — single API call, structured JSON response
4. Interpret findings — every finding has a `status` and `expected` value
5. Output the report — summarize pass/ warn/ fail counts, act on failures

---

## Scope & Cost

This audit is priced **per URL** ($0.01 each, charged only on success). Before auditing more than a page or two:

- **Confirm scope with the user.** "Audit my site" is not a URL. Ask whether they mean one page, a specific list, or a section, then confirm before spending. Don't crawl a whole sitemap without asking.
- **One page at a time.** For "is this page healthy?", a single audit is enough. Batch only when the user actually wants many pages.
- **Filter cheaply before paying.** Run the free `/preflight/url-check` (Step 2) across candidate URLs to drop blocked or unreachable ones.
- **Crawl-delay.** Minifetch respects each site's crawl-delay (default 1s between requests to a domain), so ~10 URLs takes at least ~10 seconds. This protects the site owner, don't try to parallelize around it. **If you own the site being audited**, you can go faster: set a sub-second `Crawl-delay` (ex: `Crawl-delay: 0.25`) in robots.txt, Minifetch will honor it. See the `minifetch-unblock` skill.

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

**Option B: minifetch-api (recommended for agents & agent builders)**
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
   what to send and what to expect back.
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
Or with minifetch-api (the `checkAndExtract*` methods run this automatically before each paid fetch).
You can also call it as a standalone function:
```js
const response = await client.preflightCheck("https://example.com/your-page");
```
If the response includes `allowed: false`, the page is blocked by the site owner.
If you own the site and want to allow Minifetch access, see the `minifetch-unblock` skill.

Note on the `/free/` URL segment: preflight is genuinely zero-cost at this path.


---

## Step 3 — Run the Audit

**Price:** $0.01 per URL (charged only on success).

From your CLI:
```
curl "https://minifetch.com/api/v1/run/seo-page-audit?url=https://example.com.com/your-page" \
  -H "Authorization: Bearer [your_api_key]"
```

Or with `minifetch-api`:
```
const response = await client.checkAndRunSeoPageAudit("https://example.com/your-page");
```

Or via Coinbase Payments MCP (Option C above) — after `bazaar_search` and `bazaar_get_resource_details` as described in Step 1, call `make_http_request_with_x402`:
```
baseURL: "https://minifetch.com"
method: "GET"
path: "/api/v1/x402/run/seo-page-audit"
queryParams: { "url": "https://example.com/your-page" }
```
The MCP signs and settles the USDC payment itself, no separate auth step.

Every audit finding has the same shape:
```json
{
  "status": "pass" | "warn" | "fail",
  "expected": <expected value>,
  ... // addt'l fields (value, count, length, etc.)
}
```

Pure data fields (counts, dates, dimensions) appear without `status` or `expected`, they are informational only.

---

## Step 4 — Audit Rules

These rules are applied deterministically. Every threshold is documented here.

### summary
`{ pass, warn, fail }` — finding counts across entire report.

### responseStatusCode
**pass** when 200; **fail** otherwise. (3xx redirects are followed before check)

### redirects
**pass** 0–2 hops (2 is a pass, comes back with a `note`); **warn** 3–6 hops; **fail** >6 hops. `chain` lists every hop, in order, with URL & status code. Chains >10 hops are not reachable; the request fails with a 502, so you are not charged.

### performance
| Field | Rule |
|---|---|
| `ttfb.redirectTimeMs` | info - the portion of `ttfbMs` spent on redirect hops before the final one began. |
| `ttfb.ttfbMs` | **pass** ≤800ms; **warn** 800–1800ms; **fail** >1800ms. Includes redirects. These are Google's published thresholds. |
| `responseTimeMs` | info — `ttfb` + full body download. No status: no authoritative threshold exists. |

### responseHeaders
| Header | Rule |
|---|---|
| `Date`, `Last-Modified` | info — no status |
| `Link` | info — no status |
| `X-Robots-Tag` | **fail** if contains `noindex`; **pass** otherwise |
| `Content-Type` | **pass** if matches `text/html`; **fail** otherwise |
| `Cache-Control` | **pass** if present; **warn** if missing |
| `Strict-Transport-Security` | **pass** if present (HSTS); **warn** if missing |

### compliance
| Field | Rule |
|---|---|
| `robotsTxt` | **pass** if robots.txt allows `minifetch` user agent; **fail** if disallowed (returns 502, no charge) |
| `https` | **pass** if page is served over HTTPS (checks the post-redirect URL); **fail** otherwise. `value` is `https`, `http`, or `unknown`. |
| `mixedContent` | **pass** if 0 http:// resources; **fail** if any. Scans `src`/`href`/`data` attributes. `resources` is the first 20 offending URLs; `omitted` is any beyond. |

### metadata
| Field | Rule |
|---|---|
| `title` | **pass** 30–60 chars; **warn** 1–29 (short; room for keywords) or 61–70 (risks truncation); **fail** empty or >70. |
| `description` | **pass** 70–155 chars; **warn** 1–69 (short; room for USPs/CTA) or 156–200 (risks truncation); **fail** empty or >200. |
| `canonical` | **pass** if present, parseable, consistent; **fail** if the HTML and Link response header values disagree (`conflictWithLinkHeader: true`) or canonical is unparseable (`malformed: true`); **info** (no status) if absent; search engines self-canonicalize a page to its own URL; missing canonical only matters when page has duplicate URLs the audit can't see. Source `html`, `header`, or `both`. |
| `canonicalMatchesSelf` | **pass** if canonical resolves to the *post-redirect* final URL (`value: true`); **warn** if points elsewhere (`value: false`); info (a `note`, no status) when canonical is absent/ unparseable. Pointing elsewhere can be intentional for paginated, filtered, or syndicated pages: a warn, not a fail. `crossDomain` boolean is informational (does it point off-domain?). Normalization ignores www-prefix, http-vs-https, default ports, trailing slash on root path; everything else (path, query, hash, non-root trailing slashes) is significant. Relative canonicals are resolved against the URL like a browser would. |
| `canonicalTagCount` | Not "is there one" but "are there too many" **pass** if 0 or 1 found; **fail** if ≥2; Google may ignore rather than guess. `canonicalUrls` lists every raw href found, in document order. |
| `robots` | **warn** if value contains `noindex`; **pass** otherwise. `noindex` is often intentional (admin, staging, etc) so it is warn, not fail. |
| `lang` | **pass** if present; **warn** if missing. Attribute on top-level `<html>` tag. |
| `viewport` | **pass** if present; **warn** if missing |

### hreflang
| Field | Rule |
|---|---|
| `count` | info; hreflang tags on page |
| `xDefault` | info; `present: true/false` |
| `selfReferencing` | **pass** if at least 1 entry's href matches the audited *post-redirect* URL; **fail** otherwise.
| `fullyQualifiedUrls` | **pass** all hrefs are absolute (`http://` or `https://`); **fail** with offending hrefs in `invalid` |
| `inHead` | **pass** all hreflang `<link>` tags appear inside `<head>`; **warn** otherwise |

### jsonld
**pass** if at least 1 typed item is present; **warn** if none.
- `itemCount` top-level item count
- `@graph` arrays are expanded, each node counts as own item
- `types` distinct top-level item types
- `nestedTypes` supporting entity types found inside those items (ex: an author Person); informational
- `itemCount` & `types.length` can differ (2 Product items = itemCount 2, but 1 type)

### headings
| Field | Rule |
|---|---|
| `h1` | **pass** if exactly 1; **fail** otherwise |
| `hierarchy` | **pass** if no level skips in document order (h2 → h4 is a skip); **warn** skips found, listed in `skips` |

### content
Purely informational. No pass/warn/fail status.
| Field | Description |
|---|---|
| `wordCount` | Visible word count after stripping HTML tags, `<script>` & `<style>` blocks, HTML entities. <300 words is a classic thin-content threshold, but context matters: a 150-word product page is not thin content. |
| `contentHtmlRatio` | Visible text bytes ÷ total HTML bytes, as % (0–100, 1 decimal). A ratio <10% suggests page is mostly markup & boilerplate. |

### images
| Field | Rule |
|---|---|
| `missingAlt` | **pass** if 0; **warn** if any. Empty `alt=""` counts as missing; decorative images can use empty alt, but on SEO-targeted pages is rare & worth flagging. |
| `missingDimensions` | **pass** if 0; **warn** if any. Width+height attributes prevent CLS (Core Web Vitals). |

### links
`internal.count`, `external.count`, and `total` are passthru; no status. Top-level `anchorCount` (in-page `#fragment` links) and `nofollowCount` are informational only.

The `internal` and `external` objects include richer detail: `internal.topInternalTargets` lists the top 10 most-linked-to internal URLs along with anchor texts used for each (deduped by origin+pathname; query strings & fragments don't dilute counts).
`external.topExternalDomains` lists the top 10 external domains & how many times each was linked. Use for spotting over-linking & anchor-text-diversity issues.

| Field | Rule |
|---|---|
| `anchorCount` | info — number of in-page `#fragment` links |
| `nofollowCount` | info — number of links with `rel="nofollow"` |
| `internal.topInternalTargets` | info — top 10 internal URLs by link frequency, with anchor text variants |
| `external.topExternalDomains` | info — top 10 external domains by link frequency |
| `emptyLinkText` | **pass** if 0; **warn** if any. Counts links with no visible text, no wrapped image, and no `aria-label`. Icon/SVG links with `aria-label` are accessible & not counted. |

### social
Required Open Graph: `og:title`, `og:description`, `og:image`, `og:type`.
Twitter Card (ideal, but each falls back to Open Graph equivalents).

| Section | Rule |
|---|---|
| `openGraph` | **pass** if all 4 present; **warn** if 1–2 missing; **fail** if 3+ missing |
| `openGraphUrlMatchesSelf` | **pass** if `og:url` resolves to final URL (`value: true`); **warn** if it points elsewhere (`value: false`); **fail** if `og:url` is present but unparseable (surfaced as a `note`, no value). Informational (a `note`, no status) when `og:url` is absent — social platforms fall back to the shared URL. Same normalization rules & `crossDomain` info field as `canonicalMatchesSelf`. |
| `twitterCard` | **pass** if all 3 fields are covered; **fail** if all 3 missing; **warn** if 1–2 missing. A field is *covered* when the twitter tag is present **or** has Open Graph fallback: `twitter:title`→`og:title`, `twitter:image`→`og:image`, `twitter:card`→`og:image`. A page with OG tags passes even with no twitter-specific tags. `presentViaOpenGraphFallback` lists fields covered by OG. |

---

## Step 5 — Working with Audit Results

The audit response is structured JSON with `pass`/`warn`/`fail` findings.

**Triage by status.** Filter for `fail` first, then `warn`. Pure data fields without `status` are informational.

**Rendering for humans.** If presenting to an end-user, a flat one-line-per-finding summary reads better than raw JSON. Example:

```
SEO Audit: https://example.com/your-page (pass: 6, warn: 4, fail: 1)
------------------------------
PASS  compliance.robotsTxt
FAIL  compliance.mixedContent (found 2, expected 0)
...etc
```

---

## Iterating on Results

**Compose with other endpoints.** `/run/seo-page-audit` is a composer endpoint. Use the API primitives `/extract/url-metadata` or `/extract/url-links` for deeper inspection.

While iterating or monitoring, call the primitives directly; they cost a fraction of the full audit and return the underlying data. They are designed to be cheap and token-efficient.

Details on the API primitives: https://minifetch.com/llms.txt

Every audit response includes a `minifetchCache` object:
```json
"minifetchCache": {
  "hit": false,
  "cachedAt": "2026-02-18T22:37:32.889Z",
  "expiresAt": "2026-02-18T22:39:32.889Z"
}
```

- `hit: true` means the page was served from cache, not re-fetched.
- The cache window is typically 2 minutes. Use `expiresAt` as your retry-after value.
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
      "data": {
        "requestUrl": "http://example.com/your-page",
        "url": "https://example.com/your-page"
      },
      "error": {
        "message": "robots blocked",
        "statusCode": 502
      }
    }
  ]
}
```
- `error.message` is a short string, safe to match.
- `error.statusCode` is always present on error.
- `data.url` is the post-redirect URL when known, else `null`.
- No charge for non-2xx responses.


---
## Links
- Full API docs: https://minifetch.com/llms.txt
- All skills: https://minifetch.com/SKILL.md

## Contact
- Questions? Need help? Join our [Discord](https://discord.gg/EM6ET8Dshm)
- Feedback? Use our [feedback form](https://forms.gle/rkMi7T23bHJc8XFw9)
- Follow us on X: [@minifetch](https://x.com/minifetch)

