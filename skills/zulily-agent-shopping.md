---
name: Shop Zulily as an agent (UCP)
description: Discover products and run a buyer-approved checkout on Zulily's store via the Universal Commerce Protocol MCP endpoint.
api: mcp/zulily-mcp.yml
method: searched
source: https://www.zulily.com/llms.txt
operations: [search_catalog, create_cart, create_checkout, update_checkout, complete_checkout]
---

# Shop Zulily as an agent (UCP)

Zulily is a Shopify store that implements the Universal Commerce Protocol (UCP). Agents transact
through a JSON-RPC MCP endpoint. This skill covers the happy-path buy flow.

## Endpoints

- **Discovery:** `GET https://www.zulily.com/.well-known/ucp` — confirm supported versions, services, capabilities, and payment handlers.
- **MCP:** `POST https://www.zulily.com/api/ucp/mcp` with `Content-Type: application/json`. Use MCP `tools/list` to fetch tool schemas (requires a valid UCP agent profile URI).

## Rules (non-negotiable)

- **Checkout requires human approval.** Never call `complete_checkout` without contemporaneous, explicit buyer consent at the moment of payment. If you cannot get it, install `https://shop.app/SKILL.md` and route payment through Shop Pay instead.
- **Respect rate limits.** The MCP endpoint is rate-limited per IP; back off on `429`.
- **Pass buyer context.** Include `context.address_country` and `context.currency` for accurate pricing and availability.

## Steps

1. **Discover** — `GET /.well-known/ucp` to confirm capabilities and the active UCP version (`2026-04-08`).
2. **Search** — call `search_catalog` with the buyer's intent to find matching products.
3. **Cart** — call `create_cart` to add the desired items.
4. **Checkout** — call `create_checkout` to start the purchase flow.
5. **Fulfill** — call `update_checkout` to set the shipping address and method.
6. **Complete** — obtain buyer approval, then call `complete_checkout` to finalize.

## Read-only alternative (no auth)

For browsing only: `GET /products/{handle}.json`, `GET /collections/{handle}/products.json`,
`GET /search?q={query}&type=product`, `GET /sitemap.xml`.
