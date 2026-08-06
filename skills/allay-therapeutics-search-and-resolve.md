---
name: Search Allay Therapeutics content and resolve a hit
description: >-
  Run a cross-content search against the Allay Therapeutics WordPress REST API and resolve each
  lightweight search hit back to the full post or page it points at.
api: openapi/allay-therapeutics-content-openapi.yml
operations: [searchContent, getPost, getPage, listTypes]
generated: '2026-08-06'
method: generated
---

# Search Allay Therapeutics content and resolve a hit

`https://www.allaytx.com/wp-json/wp/v2/search` searches across published posts and pages in one
call. It returns **pointers**, not documents — every hit must be resolved.

**No credentials are required.**

## 1. Search — `searchContent`

```
GET /wp/v2/search?search=ATX-101&per_page=20
```

A search for `pain` returned **25** hits at harvest (2026-08-06). Each result is a small object:

```json
{ "id": 1560, "title": "…", "url": "https://www.allaytx.com/…/", "type": "post", "subtype": "post" }
```

`X-WP-Total` carries the true match count; the `Link` header carries `rel="next"`.

## 2. Resolve each hit

Branch on `subtype`:

| `subtype` | resolve with | operation |
|-----------|--------------|-----------|
| `post` | `GET /wp/v2/posts/{id}` | `getPost` |
| `page` | `GET /wp/v2/pages/{id}` | `getPage` |

If you encounter a `subtype` you do not recognise, call `listTypes`
(`GET /wp/v2/types`) and read `rest_base` for that type to build the collection path. Only WordPress
core types are registered on this deployment (post, page, attachment, nav_menu_item, wp_block,
wp_template, wp_template_part, wp_global_styles, wp_navigation, wp_font_family, wp_font_face) — there
are no custom post types.

Cheaper alternative when you already have the id set: batch-resolve with `include`.

```
GET /wp/v2/posts?include=1560,1499&_fields=id,date,title,excerpt,link
```

## 3. Narrow the search

```
GET /wp/v2/search?search=knee&type=post&subtype=post
GET /wp/v2/search?search=knee&type=post&subtype=page
```

For a full-text search scoped to one collection, the collection's own `search` parameter is usually
better:

```
GET /wp/v2/posts?search=bupivacaine&_fields=id,date,title,link
```

## Rules

- **Search results are not documents.** `title` on a search hit is a plain string; `title.rendered`
  on the resolved object is HTML. Do not mix the two shapes.
- **Pagination:** `per_page` is bounded 1..100; over-range returns `400 rest_invalid_param`.
- **Errors:** `{code, message, data:{status}}` as `application/json`, not RFC 9457. Branch on `code`.
- **Pace:** `robots.txt` advertises `Crawl-delay: 10`. No rate-limit headers are returned, so there
  is no signal to back off on other than errors.
- **Never resolve into `/wp/v2/users`.** Author records are personal data and are deliberately
  excluded from this profile.
