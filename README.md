# Hyperice

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
