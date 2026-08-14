# Minifetch — Technical SEO Audit Rules

These rules are applied deterministically. Every threshold is documented here.

### summary
`{ pass, warn, fail }` — finding counts across entire report.

### responseStatusCode
**pass** when 200; **fail** otherwise. (3xx redirects are followed before check)

### redirects
**pass** 0–2 hops (2 is a pass, comes back with a `note`); **warn** 3–6 hops; **fail** >6 hops. `chain` lists every hop, in order, with URL & status code. Chains >10 hops fail with a 502, so you are not charged.

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
