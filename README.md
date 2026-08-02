# Hyperice

Hyperice is a recovery and movement-enhancement technology company founded in 2010 and headquartered in
Irvine, California — the Hypervolt percussion massage line, the Normatec dynamic air-compression systems it
acquired in 2020, and the Venom thermal/vibration wearables.

Hyperice runs **no developer program**: no developer portal, no docs host, no API keys, no SDKs, no OpenAPI.
What it does have is a genuine **agentic-commerce surface**, because hyperice.com runs on Shopify and has
opted into the agent layer. That surface is real, live, and machine-readable.

## API surface

| Surface | Endpoint | Status |
|---|---|---|
| Shopify Storefront GraphQL | `https://hyperice.com/api/2026-04/graphql.json` | Live. **Full introspection succeeds unauthenticated** — 424 types, 35 queries, 41 mutations. The only write surface. |
| UCP shopping server (MCP) | `https://hyperice.com/api/ucp/mcp` | Live. JSON-RPC 2.0. 13 tools. Gated behind a UCP agent profile (`422 invalid_profile_url` anonymously). |
| Storefront JSON (read-only) | `https://hyperice.com/products.json` etc. | Live, public, unauthenticated. 68 published products. |
| Customer accounts (OIDC) | `https://accounts.hyperice.com/` | OIDC + OAuth2 authorization_code + PKCE S256. |

Hyperice publishes its own agent contract at [`/llms.txt`](https://hyperice.com/llms.txt) and
[`/agents.md`](https://hyperice.com/agents.md), a UCP merchant profile at
[`/.well-known/ucp`](https://hyperice.com/.well-known/ucp), and a dedicated
`sitemap_agentic_discovery.xml` whose only entry is `/agents.md`.

## Artifacts

- `graphql/` — the full 12,465-line SDL, captured by live introspection
- `openapi/` — an API Evangelist **derivation** of the public read surface (Hyperice publishes no OpenAPI)
- `mcp/` — the MCP server manifest and the REST/GraphQL/MCP tool crosswalk
- `llms/` — `/llms.txt` and `/agents.md`, verbatim
- `well-known/` — UCP profile, OIDC discovery, RFC 8414 and RFC 9728 metadata, verbatim
- `authentication/`, `scopes/` — the three-tier auth posture, read from live discovery documents
- `conventions/`, `errors/`, `rate-limits/`, `lifecycle/`, `data-model/`, `conformance/`
- `skills/`, `agentic-access/`, `overlays/`, `packages/`, `security/`

## Notable findings

- **`api.`, `developer.`, `docs.`, `mcp.`, `status.` and `trust.hyperice.com` all answer HTTP 200 and none of
  them exists.** A wildcard AmazonS3 record 301-redirects every unprovisioned subdomain to the storefront
  homepage. There is no developer portal, no status page and no trust center.
- The Storefront GraphQL API is served in **public access mode**, so every cart and customer mutation is
  reachable with no credential — the widest agentic exposure on this perimeter, and entirely undocumented by
  Hyperice, whose `/llms.txt` describes only the read-only surface and the UCP tools.
- No A2A agent card (`/.well-known/agent-card.json` and `/.well-known/agent.json` both 404), no
  `security.txt`, no compliance program, no webhooks or AsyncAPI, and no first-party SDKs.
- The GitHub org `hyper-ice` is **not** this company (blog `moe.hyice.org`, zero public repos).

- https://hyperice.com/
- https://forgeglobal.com/hyperice_stock/
