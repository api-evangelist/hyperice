---
name: Browse the Hyperice product catalog
description: Read products and collections from Hyperice's unauthenticated storefront surfaces, picking the right projection for the job.
api: openapi/hyperice-storefront-openapi.yml
operations: [listProducts, getProduct, listCollections, listCollectionProducts]
generated: '2026-07-31'
method: generated
---

# Browse the Hyperice product catalog

Hyperice sells recovery hardware (Hypervolt percussion massagers, Normatec
compression systems, Venom thermal wearables) from a Shopify storefront at
`https://hyperice.com`. The catalog is fully readable **with no credential of any
kind**. Hyperice states this itself in `/agents.md` under "Read-Only Browsing (No
Authentication Required)".

## Authentication

None. Do not send an API key, a bearer token, or an
`X-Shopify-Storefront-Access-Token` — no key exists and none is required.

## Pick the right surface

There are three read projections and they are not equivalent:

| Need | Use |
|---|---|
| Whole catalog, simple fields | REST `listProducts` — `GET /products.json` |
| One product you know the handle of | REST `getProduct` — `GET /products/{handle}.json` |
| **Keyword search** | GraphQL `search(query:)` or `predictiveSearch(query:)` |
| Selected fields only / nested data | GraphQL `products`, `product`, `collection` |

The REST surface **cannot search** — `/products.json` accepts no query term. If
the task is "find the massage gun under $200", you must use GraphQL or the
HTML `GET /search?q={query}&type=product` route.

## Steps

1. **List collections** with `listCollections` — `GET /collections.json`.
   Each entry gives `handle`, `title`, `description` and `products_count`.

2. **List products**, either across the store with `listProducts`
   (`GET /products.json`) or within one collection with `listCollectionProducts`
   (`GET /collections/{handle}/products.json`).

   Paginate with `limit` (max **250**, default 50) and `page` (1-indexed).
   At the time of writing the store publishes **68** products, so
   `?limit=250` returns everything in one call.

   There is **no** total count, **no** `next` link and **no** `Link` header.
   You know you have reached the end when a page comes back shorter than
   `limit`.

3. **Fetch one product** with `getProduct` — `GET /products/{handle}.json`.
   The response is `{"product": {...}}`.

## Reading the response

- Price lives on the **variant**, not the product: `variants[].price` is a
  decimal **string** in the store currency (USD). `compare_at_price` is the
  strike-through price and is often `null`.
- `variants[].available` is the stock flag. Never present an unavailable
  variant as purchasable.
- `handle` is the stable join key across every surface. Use it, not `id` —
  the REST surface returns bare integer ids while GraphQL returns base64
  `gid://shopify/Product/...` global ids, and **the two are not
  interchangeable**.
- `body_html` is HTML, not plain text. Strip tags before showing it to a user.
- `options[]` gives the option axes (e.g. colour) and `variants[].option1..3`
  map positionally onto them.

## Errors

- A missing handle returns **HTTP 404 with a zero-length body** and
  `content-type: application/json`. There is no error document — do not try to
  parse the body.
- Unpublished products are simply absent from these endpoints; a 404 does not
  prove the product never existed.

## Rate limits

Hyperice publishes no numeric limit for the REST surface and returns no
`X-RateLimit-*` or `Retry-After` headers. Be conservative: prefer one
`?limit=250` call over 68 individual fetches.

## Localization

The store serves 17 locales via URL prefix (`/en-gb/`, `/de-de/`, `/fr-ca/`, …).
Prices and availability differ by locale. Hyperice's guidance is explicit:
"Use buyer context. Pass `context.address_country` and `context.currency` for
accurate pricing and availability."

## Related

- `conventions/hyperice-conventions.yml` — pagination, errors, versioning
- `data-model/hyperice-data-model.yml` — the entity graph
- `skills/hyperice-agent-commerce-checkout.md` — buying, not just browsing
