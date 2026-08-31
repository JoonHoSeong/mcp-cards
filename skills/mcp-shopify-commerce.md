# Shopify Commerce & Storefront MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Shopify Commerce & Storefront MCP Server |
| **Category** | E-Commerce/Platform |
| **Official** | ⭐ Official (Shopify) |
| **Source** | https://github.com/shopify/mcp-shopify |
| **Transport** | stdio |
| **Install** | `npx -y @shopify/mcp-server` |

## Tools & Capabilities
- `query_products` — Search store catalog, product variants, inventory levels, and prices
- `manage_orders` — Query order fulfillments, payment status, customer addresses, and refunds
- `create_draft_order` — Scaffold custom draft orders with applied discounts and line items
- `execute_graphql_admin` — Run custom Shopify Admin GraphQL queries and mutations

## Client Configuration
```json
{
  "mcpServers": {
    "shopify": {
      "command": "npx",
      "args": [
        "-y",
        "@shopify/mcp-server"
      ],
      "env": {
        "SHOPIFY_SHOP_DOMAIN": "your-store.myshopify.com",
        "SHOPIFY_ADMIN_ACCESS_TOKEN": "shpat_..."
      }
    }
  }
}
```

## Security & Best Practices
- Admin API Access Token with restricted OAuth scopes. Customer PII masking.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
