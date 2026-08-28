# Paddle MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | paddle-mcp-server |
| **Category** | Payments/Billing |
| **Official** | Yes (Paddle) |
| **Source** | https://github.com/PaddleHQ/paddle-mcp-server |
| **Transport** | stdio |
| **Install** | `npm install @paddle/mcp-server` |

## Description
Paddle's official MCP server provides AI agents with access to Paddle's Merchant of Record (MoR) billing platform. As a complete payments infrastructure that handles sales tax, VAT, and compliance globally, it enables agents to manage products, subscriptions, transactions, and revenue reporting without the merchant handling tax obligations.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_products` | List catalog products with filters |
| `create_product` | Create a new product in the catalog |
| `list_prices` | List pricing configurations |
| `list_subscriptions` | List active and past subscriptions |
| `create_transaction` | Create a new billable transaction |
| `get_revenue_report` | Fetch revenue and tax reports |
| `list_customers` | List customers with search |
| `manage_discounts` | Create, update, or expire discount codes |

## Configuration
```json
{
  "mcpServers": {
    "paddle": {
      "command": "npx",
      "args": ["@paddle/mcp-server"],
      "env": {
        "PADDLE_API_KEY": "your-api-key",
        "PADDLE_ENVIRONMENT": "sandbox"
      }
    }
  }
}
```

## Use Cases
1. SaaS billing agent managing subscriptions with automatic tax handling
2. Revenue reporting and analytics for developer tool companies
3. Discount and promotion management for product launches

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| SaaS monetization | aiq-deploy | Monetize deployed AI agents with Paddle subscriptions |
| Revenue analytics | nemotron-reasoning | Analyze subscription metrics and predict churn |
| Pricing optimization | portfolio-optimization | Optimize pricing tiers based on usage data |

## Prerequisites
- Paddle seller account
- `PADDLE_API_KEY` — API key from Paddle dashboard
- Node.js 18+

## Security Notes
- Use sandbox environment for development
- Verify Paddle webhook signatures (SHA256)
- Paddle handles PCI compliance as Merchant of Record
- Restrict API key to necessary permissions

## References
- https://github.com/PaddleHQ/paddle-mcp-server
- https://developer.paddle.com/api-reference/overview
- https://developer.paddle.com/concepts/overview
