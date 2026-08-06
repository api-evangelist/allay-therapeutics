---
name: Read the Allay Therapeutics pipeline and science pages
description: >-
  Pull the Allay Therapeutics corporate pages — Our Science, Pipeline, About Us, Contact Us — as
  structured JSON from the WordPress REST content API instead of scraping rendered HTML.
api: openapi/allay-therapeutics-content-openapi.yml
operations: [listPages, getPage]
generated: '2026-08-06'
method: generated
---

# Read the Allay Therapeutics pipeline and science pages

The nine corporate pages behind `www.allaytx.com` are readable as JSON at
`https://www.allaytx.com/wp-json`. Use this instead of scraping the rendered site.

**No credentials are required.**

## 1. List the pages — `listPages`

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,title,link,modified
```

Nine pages are published. At harvest (2026-08-06):

| id | slug | title |
|----|------|-------|
| 632 | `home` | Home |
| 636 | `about-us` | About Us |
| 644 | `our-science` | Our Science |
| 653 | `pipeline` | Pipeline |
| 661 | `news` | News |
| 664 | `careers` | Careers |
| 666 | `contact-us` | Contact Us |
| 937 | `privacy-policy` | Privacy Notices |
| 941 | `terms-of-service` | Terms of Service |

Resolve by slug rather than hard-coding the id:

```
GET /wp/v2/pages?slug=pipeline&_fields=id,slug,title,content,link
```

The page tree is flat — `parent` is `0` on every page.

## 2. Fetch one page — `getPage`

```
GET /wp/v2/pages/653
```

`content.rendered` is the full page HTML. On this deployment it is block-editor markup, so headings
and tables survive an HTML-to-text conversion; do not assume plain text.

## What lives where

- **`our-science` (644)** — the tunable drug-biopolymer delivery platform: validated non-opioid
  analgesics combined with dissolvable biopolymers to release relief at a targeted site over weeks
  rather than days, tunable for constant release (chronic pain such as osteoarthritis) or pulsed
  release (cyclic pain such as gout).
- **`pipeline` (653)** — the program table: **ATX-101** (bupivacaine implant for total knee
  replacement pain), ATX-201 (new formulations), ATX-301 (injectables), ATX-401 (an additional
  clinical indication), ATX-501 (on-demand anesthetic delivery).
- **`about-us` (636)** — corporate positioning.
- **`contact-us` (666)** — the San Jose, CA and Singapore office addresses and the
  `contact@allaytx.com` / `careers@allaytx.com` inboxes.

## Rules

- **Freshness:** compare `modified` against your last run rather than refetching everything. The
  Yoast page sitemap reported a `lastmod` of 2026-03-18.
- **Caching:** responses carry `cache-control: max-age=600, must-revalidate` and are served through
  a WP Engine cache. Honour it.
- **Errors:** a bad id returns `404 rest_post_invalid_id` in the WordPress envelope
  `{code, message, data:{status}}` — not RFC 9457.
- **Do not treat this as a product API.** It is the marketing site's CMS surface. Allay Therapeutics
  publishes no developer program, no SLA, no status page and no deprecation policy; the contract can
  change whenever WordPress or a plugin is updated.
