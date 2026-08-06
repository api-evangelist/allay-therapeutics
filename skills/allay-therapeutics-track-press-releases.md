---
name: Track Allay Therapeutics press releases
description: >-
  Read the Allay Therapeutics news archive from its WordPress REST content API — list press
  releases, filter by category or date window, and fetch a single announcement in full.
api: openapi/allay-therapeutics-content-openapi.yml
operations: [listCategories, listPosts, getPost]
generated: '2026-08-06'
method: generated
---

# Track Allay Therapeutics press releases

Allay Therapeutics is a clinical-stage biopharmaceutical company developing ultra-sustained,
non-opioid analgesics for post-surgical pain (lead candidate **ATX-101**, a bupivacaine implant for
total knee replacement). Its corporate news archive is readable over the WordPress REST API at
`https://www.allaytx.com/wp-json`.

**No credentials are required.** Do not send an `Authorization` header — every operation below is
anonymous.

## 1. Resolve the category you want — `listCategories`

```
GET /wp/v2/categories?per_page=100
```

Four categories are registered. At harvest (2026-08-06):

| id | slug | name | posts |
|----|------|------|-------|
| 5 | `press-releases` | Press Releases | 14 |
| 10 | `in-the-news` | In The News | 4 |
| 23 | `presentations-and-publications` | Presentations and Publications | 1 |
| 1 | `uncategorized` | Uncategorized | 0 |

Resolve the id at runtime rather than hard-coding it — counts and ids can move.

## 2. List the archive — `listPosts`

```
GET /wp/v2/posts?categories=5&per_page=100&orderby=date&order=desc&_fields=id,date,slug,title,link,categories
```

- The whole archive is **19 posts**, 2021-05-13 through 2025-06-05. One request at
  `per_page=100` covers it.
- Read **`X-WP-Total`** and **`X-WP-TotalPages`** from the response headers for the true count, and
  follow the `Link: …; rel="next"` header rather than incrementing `page` blindly.
- Use `_fields` to trim the payload — the default representation inlines the full rendered HTML plus
  a large `yoast_head` block.

Filter to a window with ISO 8601 bounds:

```
GET /wp/v2/posts?after=2025-01-01T00:00:00&before=2026-01-01T00:00:00&_fields=id,date,title,link
```

## 3. Fetch one announcement — `getPost`

```
GET /wp/v2/posts/{id}
```

`title.rendered`, `excerpt.rendered` and `content.rendered` are HTML strings — strip or render them,
do not treat them as plain text. `link` is the canonical public permalink; prefer it when citing.

## Rules

- **Pagination:** `per_page` is bounded 1..100. Anything higher returns `400 rest_invalid_param`
  with the failing parameter named in `data.params`. Clamp, do not retry the same value.
- **Errors:** the envelope is `{code, message, data:{status}}` as `application/json` — *not* RFC 9457
  problem details. Branch on `code`, not on `message`. See
  `errors/allay-therapeutics-problem-types.yml`.
- **A 404 outside `/wp-json/` is HTML,** not the JSON envelope. Check `content-type` before parsing.
- **No idempotency contract and no rate-limit headers.** Every operation here is a GET. `robots.txt`
  advertises `Crawl-delay: 10` — pace accordingly.
- **Never fetch `/wp/v2/users`.** It answers anonymously on this deployment, but author records are
  personal data and this profile deliberately excludes them.
