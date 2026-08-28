# Lemon Squeezy MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | lemonsqueezy-mcp |
| **Category** | Payments/Billing |
| **Official** | No (Community — YawLabs) |
| **Source** | https://github.com/YawLabs/lemonsqueezy-mcp |
| **Transport** | stdio |
| **Install** | `npm install lemonsqueezy-mcp` |

## Description
A community-built MCP server for Lemon Squeezy, the all-in-one platform for selling digital products, subscriptions, and software licenses. Designed for indie hackers and solo developers, it enables AI agents to manage products, orders, subscriptions, license keys, and revenue — with Lemon Squeezy handling tax, payments, and compliance as Merchant of Record.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_products` | List all digital products in store |
| `list_orders` | List orders with status filters |
| `list_subscriptions` | List active subscriptions |
| `create_checkout` | Generate a checkout URL for a product |
| `list_license_keys` | List issued software license keys |
| `activate_license` | Activate a license key for a user |
| `list_customers` | List customers with search |
| `get_revenue` | Fetch revenue and sales metrics |

## Configuration
```json
{
  "mcpServers": {
    "lemonsqueezy": {
      "command": "npx",
      "args": ["lemonsqueezy-mcp"],
      "env": {
        "LEMONSQUEEZY_API_KEY": "your-api-key"
      }
    }
  }
}
```

## Use Cases
1. Indie developer agent managing digital product sales and license distribution
2. Automated license key activation and validation for software products
3. Revenue dashboard agent tracking sales across multiple digital products

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Sell synthetic data | data-designer | Monetize AI-generated synthetic datasets via Lemon Squeezy |
| Model marketplace | nemotron-customize | Sell fine-tuned model access with license keys |
| Sales analytics | nemotron-reasoning | Analyze product performance and conversion rates |

## Prerequisites
- Lemon Squeezy store account
- `LEMONSQUEEZY_API_KEY` — API key from store settings
- Node.js 18+

## Security Notes
- Community-maintained; review source before production use
- Verify webhook signatures for order notifications
- Use separate API keys for different environments
- Lemon Squeezy handles PCI compliance as MoR

## References
- https://github.com/YawLabs/lemonsqueezy-mcp
- https://docs.lemonsqueezy.com/api
- https://www.lemonsqueezy.com/
