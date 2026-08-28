# Mercado Pago MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mercadopago-mcp |
| **Category** | Payments/Billing |
| **Official** | Yes (Mercado Pago) |
| **Source** | https://mcp.mercadopago.com |
| **Transport** | stdio |
| **Install** | `npm install @mercadopago/mcp-server` |

## Description
Mercado Pago's official MCP server enables AI agents to process payments across Latin America's largest payment ecosystem. It supports local payment methods including PIX (Brazil), OXXO (Mexico), and regional credit/debit cards, with built-in support for subscriptions, preferences, and refund management.

## Key Tools
| Tool | Description |
|------|-------------|
| `create_payment` | Process a payment with local methods |
| `create_preference` | Create a checkout preference with items |
| `list_payments` | List payments with date/status filters |
| `get_payment_status` | Check real-time payment status |
| `create_subscription` | Set up recurring billing |
| `refund_payment` | Issue full or partial refund |

## Configuration
```json
{
  "mcpServers": {
    "mercadopago": {
      "command": "npx",
      "args": ["@mercadopago/mcp-server"],
      "env": {
        "MP_ACCESS_TOKEN": "your-access-token"
      }
    }
  }
}
```

## Use Cases
1. E-commerce agent processing payments in Brazil, Mexico, Argentina, and Colombia
2. Subscription management for Latin American SaaS products
3. Payment status monitoring and automated refund processing

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Retail commerce | retail-agentic-commerce | LatAm retail checkout with local payment methods |
| Customer insights | nemotron-reasoning | Analyze payment patterns across regions |
| Fraud screening | financial-fraud-detection | Screen high-risk LatAm transactions |

## Prerequisites
- Mercado Pago seller account
- `MP_ACCESS_TOKEN` — Production or sandbox access token
- Node.js 18+

## Security Notes
- Use test credentials for sandbox environment
- Rotate access tokens periodically
- Implement IPN (Instant Payment Notification) verification
- Restrict token permissions to minimum required scopes

## References
- https://mcp.mercadopago.com
- https://www.mercadopago.com.br/developers/en/docs
- https://www.mercadopago.com.br/developers/en/reference
