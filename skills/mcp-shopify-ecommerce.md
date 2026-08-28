# Shopify MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Shopify MCP |
| **Category** | E-commerce |
| **Official** | Community |
| **Source** | https://github.com/benwmerritt/shopify-mcp |
| **Transport** | stdio |
| **Install** | `npx shopify-mcp` |

## Description
The Shopify MCP server provides AI agents with comprehensive access to Shopify stores, offering 30+ tools for managing products, orders, customers, inventory, and discounts. It enables end-to-end e-commerce automation and intelligent catalog management.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_products` | List products with filters |
| `create_product` | Create a new product listing |
| `list_orders` | List recent orders |
| `get_order` | Get detailed order information |
| `update_inventory` | Update stock levels |
| `list_customers` | List customer records |
| `create_discount` | Create discount codes or rules |
| `list_collections` | List product collections |

## Configuration
```json
{
  "mcpServers": {
    "shopify": {
      "command": "npx",
      "args": ["shopify-mcp"],
      "env": {
        "SHOPIFY_ACCESS_TOKEN": "${SHOPIFY_ACCESS_TOKEN}",
        "SHOPIFY_STORE_URL": "${SHOPIFY_STORE_URL}"
      }
    }
  }
}
```

## Use Cases
1. AI-powered product catalog management and enrichment
2. Automated order processing and customer communication
3. Dynamic pricing and inventory optimization

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Agentic commerce | retail-agentic-commerce | AI-driven shopping experiences with product recommendations and conversational commerce |
| Catalog enrichment | retail-catalog-enrichment | Automated product descriptions, categorization, and attribute extraction |

## Prerequisites
- `SHOPIFY_ACCESS_TOKEN` — Shopify Admin API access token
- `SHOPIFY_STORE_URL` — Store URL (e.g., `mystore.myshopify.com`)

## Security Notes
- Access token provides store management capabilities; use minimum required scopes
- Order and customer data contains PII; handle according to privacy regulations

## References
- https://github.com/benwmerritt/shopify-mcp
- https://shopify.dev/docs/api/admin
