---
name: minifetch-unblock
description: "Configure a site's robots.txt to allow (or block) the Minifetch crawler. Use when a site owner wants Minifetch to fetch their pages — e.g. after a preflight check returns allowed:false — or wants to grant access on specific paths, set or speed up the crawl-delay, or block Minifetch entirely."
---
# Minifetch Skill: Allow Minifetch Access via robots.txt

This skill is for **site owners** who want to explicitly allow Minifetch to fetch their pages while working with the Minifetch API.

**Base URL:** https://minifetch.com

---

## How Minifetch identifies itself

Minifetch checks your robots.txt before fetching its pages using your site's user agent rules. It identifies itself with the following user agent string:

```
minifetch/1.0 (+https://minifetch.com/site-owner-faq)
```

Minifetch matches on the `minifetch` token, so any `User-agent` directive containing `minifetch` (case-insensitive) will be picked up correctly.

- If your robots.txt is missing or returns an error, Minifetch defaults to **allowed**.
- If it returns a status code 403, 418, or 429, Minifetch treats the entire site as **blocked**.
- Paid API requests to that URL return 502 (no charge).

---

## Allow Minifetch while blocking all other bots

Add the following to your `robots.txt`. The order of blocks does not matter. Minifetch's parser matches on the most specific user-agent rule:

```
User-agent: minifetch
Allow: /

User-agent: *
Disallow: /
```

This explicitly grants Minifetch access to all pages while blocking every other crawler.

---

## Allow Minifetch on specific paths only

To restrict Minifetch to certain sections of your site:

```
User-agent: minifetch
Allow: /blog/
Allow: /products/
Disallow: /

User-agent: *
Disallow: /
```

---

## Block Minifetch entirely

To block Minifetch along with all other bots:

```
User-agent: *
Disallow: /
```

Or to block Minifetch specifically while allowing other bots:

```
User-agent: minifetch
Disallow: /

User-agent: *
Allow: /
```

---

## Set a crawl delay

Minifetch strictly observes the `Crawl-delay` directive (value in seconds). Use it to slow Minifetch down — or to **speed it up** below the default so audits of your own site finish faster. For example, to let Minifetch fetch your pages twice as fast as the default:

```
User-agent: minifetch
Allow: /
Crawl-delay: 0.5
```

Here `Crawl-delay: 0.5` tells Minifetch it may fetch a page every half-second instead of the default one second — halving the time to crawl a batch of your URLs. Fractional (sub-second) values are honored. Without any `Crawl-delay` set, Minifetch defaults to 1 second between requests.

---

## Verify your robots.txt

After updating your robots.txt, you can verify Minifetch can fetch your pages correctly using the free preflight endpoint:

From your cli:
```
curl "https://minifetch.com/api/v1/free/preflight/url-check?url=https://example.com/your-page"
```

Or using [minifetch-api](https://www.npmjs.com/package/minifetch-api):
```js
await client.preflightCheck("https://example.com/your-page");
```

A successful response will show:
```json
{
  "success": true,
  "results": [
    {
      "data": {
        "url": "https://example.com/your-page",
        "allowed": true,
        "crawlDelay": 1
      }
    }
  ]
}
```

If `allowed` is still `false` after updating, check that your robots.txt is accessible at `https://example.com/robots.txt` and has been re-deployed. Minifetch caches robots.txt for 24 hours, so changes may take up to a day to propagate.

---

## Questions?

Visit our [Site Owner FAQ](https://minifetch.com/site-owner-faq) for more detail on how Minifetch identifies itself, what it does and does not crawl, and our scraping practices.


---
## Links
- Full API docs: https://minifetch.com/llms.txt
- All skills: https://minifetch.com/SKILL.md

## Contact
- Questions? Need help? Join our [Discord](https://discord.gg/EM6ET8Dshm)
- Feedback? Use our [feedback form](https://forms.gle/rkMi7T23bHJc8XFw9)
- Follow us on X: [@minifetch](https://x.com/minifetch)

