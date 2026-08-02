---
name: Buy a Hyperice product as an agent
description: Run the six-step UCP flow Hyperice publishes in /llms.txt, from catalog discovery through a buyer-approved checkout.
api: mcp/hyperice-mcp.yml
operations: [search_catalog, lookup_catalog, create_cart, update_cart, create_checkout, update_checkout, complete_checkout]
generated: '2026-07-31'
method: generated
---

# Buy a Hyperice product as an agent

Hyperice implements the **Universal Commerce Protocol (UCP)** and runs a live
MCP shopping server. This skill follows the exact flow Hyperice documents in
`https://hyperice.com/llms.txt`.

## Before you start: two hard rules from Hyperice

1. **Checkout requires human approval.** Hyperice's own words: *"Agents must not
   complete payment without explicit buyer consent. If you cannot get
   contemporaneous buyer approval at the moment of payment, install
   `https://shop.app/SKILL.md` and route the purchase through Shop Pay
   instead."* Never call `complete_checkout` on a schedule, in a retry loop, or
   without the buyer present.

2. **Respect rate limits.** The MCP endpoint is rate-limited per IP. Back off on
   429.

Hyperice additionally recommends that general-purpose personal shopping
assistants use the cross-store Shop skill (`https://shop.app/SKILL.md`) rather
than driving this endpoint directly. Prefer that unless you specifically need
store-level control.

## Endpoint and access

- MCP endpoint: `POST https://hyperice.com/api/ucp/mcp`
  (canonical origin: `https://hypericeprod.myshopify.com/api/ucp/mcp`)
- Headers: `Content-Type: application/json`,
  `Accept: application/json, text/event-stream`
- Protocol: MCP over JSON-RPC 2.0
- UCP version: `2026-04-08` (also accepts `2026-01-23`)

**You must present a resolvable UCP agent profile URI.** Without one the server
refuses everything, including `tools/list`, with:

```
HTTP 422
{"jsonrpc":"2.0","id":1,"error":{"code":-32001,"message":"UCP discovery failed",
 "data":{"code":"invalid_profile_url","content":"Unable to fetch agent profile: Missing profile uri",
 "continue_url":"https://hypericeprod.myshopify.com/"}}}
```

If you see `invalid_profile_url`, the fix is to publish an agent profile — not to
retry. Note the `continue_url` field: it is the human-recovery path to hand back
to the buyer.

## Every call takes `meta`

Each UCP method has a required `meta` parameter. Two fields matter most:

- **`idempotency-key`** — *"Unique key for retry safety. Maps to HTTP
  Idempotency-Key header."* Generate a fresh key per distinct intent and reuse it
  **only** to retry the byte-identical request. This is what makes
  `complete_checkout` safe to retry after a timeout.
- **buyer context** — pass `context.address_country` and `context.currency`.
  Prices and availability are locale-dependent across 17 storefront locales.

## The flow

1. **Discover** — `GET https://hyperice.com/.well-known/ucp` and confirm the
   capabilities you need are present. Expect
   `dev.ucp.shopping.catalog.search`, `.cart`, `.checkout`, `.fulfillment`,
   `.discount`, `.order`.

2. **Search** — `search_catalog` to find products matching the buyer's intent.
   Use `lookup_catalog` (batch, by identifier) or `get_product` when you already
   know what you want.

3. **Cart** — `create_cart` with the chosen variants. Amend with `update_cart`.

4. **Checkout** — `create_checkout` to start the purchase.

5. **Fulfill** — `update_checkout` to set the shipping address and delivery
   method. Note the fulfillment capability config: this store sets
   `allows_multi_destination.shipping: false`, so **one destination per order** —
   do not attempt to split a shipment.

6. **Complete** — `complete_checkout`, **only** with contemporaneous buyer
   approval.

## Payment

The store advertises three payment handlers. You do not handle card data
yourself:

| Handler | Id | Notes |
|---|---|---|
| `com.google.pay` | `gpay` | VISA, MASTERCARD, AMEX, DISCOVER; full billing address + phone required |
| `dev.shopify.card` | `shopify.card` | visa, master, american_express, discover, diners_club |
| `dev.shopify.shop_pay` | `shop_pay` | Shop Pay |

## Failure handling

- **`invalid_profile_url` (422)** — your agent profile is missing or
  unresolvable. Not retryable.
- **429** — back off exponentially.
- **Card failures** — see `errors/hyperice-decline-codes.yml`. Hyperice is a
  merchant, not an acquirer: no issuer decline code, AVS result or CVV result is
  exposed. The most specific signal you get is
  `PAYMENTS_CREDIT_CARD_GENERIC`, so you **cannot** distinguish "insufficient
  funds" from "issuer declined". Never auto-retry a declined authorization —
  return to the buyer and ask for a different instrument.
- **Validation failures** — see `errors/hyperice-problem-types.yml` for the 151
  enumerated codes.

## What this surface cannot do

Recorded in `mcp/hyperice-tool-crosswalk.yml`:

- There is **no collection/category tool**. Catalog access is search-and-lookup
  shaped. Browse collections over REST or GraphQL instead.
- `get_order` needs the OIDC customer-account tier
  (`customer-account-mcp-api:full` at `accounts.hyperice.com`), not the anonymous
  agent identity used above.
- `cancel_checkout` has no counterpart on any other Hyperice surface.

## Related

- `mcp/hyperice-mcp.yml` — the server manifest and tool list
- `mcp/hyperice-tool-crosswalk.yml` — how these tools map to GraphQL and REST
- `agentic-access/hyperice-agentic-access.yml` — consequence classification
- `skills/hyperice-graphql-cart.md` — the GraphQL fallback path
