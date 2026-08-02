---
name: Build and submit a cart on the Storefront GraphQL API
description: Use Hyperice's Shopify Storefront GraphQL surface to build a cart and submit it for completion, handling the two-channel error model and union-typed results.
api: graphql/hyperice-storefront.graphql
operations: [cartCreate, cartLinesAdd, cartLinesUpdate, cartBuyerIdentityUpdate, cartDeliveryAddressesAdd, cartSelectedDeliveryOptionsUpdate, cartPrepareForCompletion, cartSubmitForCompletion]
generated: '2026-07-31'
method: generated
---

# Build and submit a cart on the Storefront GraphQL API

Use this when you need control the UCP tools do not give you (collections,
blog/CMS content, customer accounts, selected-field queries), or when the UCP
agent-profile gate is not available to you.

## Endpoint

```
POST https://hyperice.com/api/2026-04/graphql.json
Content-Type: application/json
```

**No authentication.** This store serves the Storefront API in public access
mode — full introspection succeeds anonymously. Do not send an
`X-Shopify-Storefront-Access-Token`.

The complete schema is captured in `graphql/hyperice-storefront.graphql`
(424 types, 35 queries, 41 mutations). **Grep it before writing any query** —
do not guess field names.

## Versioning — check this first

The version is a date in the path. Supported at capture time:
`2025-10`, `2026-01`, `2026-04`, `2026-07`. Confirm at runtime:

```graphql
query { publicApiVersions { handle supported displayName } }
```

A retired version fails hard with **HTTP 404** and
`{"errors":[{"message":"Not Found","extensions":{"code":"NOT_FOUND"}}]}`.
There is no `Sunset` header and no deprecation warning — pin a supported version
and re-check it periodically. Every response echoes `x-shopify-api-version`.

## The critical model difference

**There is no `Checkout` type in this schema.** The cart *is* the checkout. UCP's
`create_checkout` / `complete_checkout` map onto cart mutations, not onto a
separate checkout object.

## Steps

1. **`cartCreate`** — create the cart. Keep the returned cart `id`.
2. **`cartLinesAdd`** — add merchandise lines by variant global id.
   `cartLinesAdd` is **append**-style and therefore **not** safe to blind-retry;
   `cartLinesUpdate` is set-style and is.
3. **`cartBuyerIdentityUpdate`** — attach email, phone and `countryCode`.
4. **`cartDeliveryAddressesAdd`** — attach the shipping address.
5. **`cartSelectedDeliveryOptionsUpdate`** — pick a delivery option from the
   cart's `deliveryGroups`.
6. **`cartPrepareForCompletion`** — returns a union:
   `CartStatusReady` | `CartStatusNotReady` | `CartThrottled`.
7. **`cartSubmitForCompletion`** — returns a union:
   `SubmitSuccess` | `SubmitFailed` | `SubmitAlreadyAccepted` | `SubmitThrottled`.
   **This places a real order. Get explicit buyer approval first** — Hyperice's
   published rule applies on this surface too.

## Two error channels — you must check both

GraphQL failures on this API arrive in **two different places**, and both come
back as **HTTP 200**:

| Channel | Where | Shape |
|---|---|---|
| Transport / validation | top-level `errors[]` | `{message, locations[], path[], extensions{code, typeName, fieldName}}` |
| Domain / user error | `data.<mutation>.userErrors[]` | `{field[], message, code}` |

An integration that only checks `errors[]` will treat a rejected cart mutation as
a success. Typed user-error types: `CartUserError`, `CustomerUserError`,
`MetafieldsSetUserError`, `MetafieldDeleteUserError`,
`UserErrorsShopPayPaymentRequestSessionUserError`.

Enumerated codes are catalogued in `errors/hyperice-problem-types.yml`:
58 `CartErrorCode`, 95 `SubmissionErrorCode`, 15 `CustomerErrorCode`,
10 `MetafieldsSetUserErrorCode`.

## Throttling is a union member, not a 429

There is **no HTTP 429** on this surface. Throttling arrives as `CartThrottled`
(from `cartPrepareForCompletion`) or `SubmitThrottled` (from
`cartSubmitForCompletion`) inside a 200 response. Branch on the union
`__typename`, not the status code.

Every response carries `extensions.cost.requestedQueryCost` — use it to keep
queries small.

If you call from a server, send the case-sensitive
`Shopify-Storefront-Buyer-IP` header so per-IP throttling attributes to the
buyer rather than your egress IP. The schema warns that the value must be
trusted: *"Unthrottled access to this mutation presents a security risk."*

## Idempotency

Cart mutations are **not** idempotency-keyed. The only mutation with a required
idempotency key is `shopPayPaymentRequestSessionSubmit(idempotencyKey: String!)`,
which raises `IDEMPOTENCY_KEY_ALREADY_USED` on a conflicting replay. If you need
retry-safe order placement, prefer the UCP path
(`skills/hyperice-agent-commerce-checkout.md`), where every method accepts
`meta['idempotency-key']`.

## Pagination

Relay cursor connections throughout: `first`/`last` with `after`/`before`,
returning `edges { node cursor }` and
`pageInfo { hasNextPage hasPreviousPage startCursor endCursor }`.

## Identifiers

GraphQL uses base64 global ids (`gid://shopify/ProductVariant/...`). The REST
`.json` surface uses bare integers. **They are not interchangeable.** Join on
`handle` when crossing surfaces.

## Related

- `graphql/hyperice-storefront.graphql` — the full SDL
- `conventions/hyperice-conventions.yml` — errors, pagination, versioning
- `errors/hyperice-problem-types.yml` — the 151 enumerated codes
- `data-model/hyperice-data-model.yml` — the cart-to-order lifecycle path
