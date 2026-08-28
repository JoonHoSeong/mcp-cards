# Adyen MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | adyen-mcp |
| **Category** | Payments/Billing |
| **Official** | Yes (Adyen) |
| **Source** | https://github.com/Adyen/adyen-mcp |
| **Transport** | stdio |
| **Install** | `npm install @adyen/mcp-server` |

## Description
Adyen's official MCP server enables AI agents to manage the full payment lifecycle across Adyen's global infrastructure. Supporting 250+ payment methods across Europe, APAC, and globally, it provides tools for payment processing, payouts, terminal management, webhook configuration, and balance inquiries — ideal for enterprise-grade commerce.

## Key Tools
| Tool | Description |
|------|-------------|
| `create_payment` | Initiate a payment with 250+ methods |
| `list_payments` | List payments with merchant-level filters |
| `get_payment_details` | Get detailed payment information and status |
| `create_payout` | Send funds to sellers or partners |
| `list_terminals` | List POS terminals and their status |
| `configure_webhook` | Set up or update webhook endpoints |
| `get_balance` | Check merchant account balance |

## Configuration
```json
{
  "mcpServers": {
    "adyen": {
      "command": "npx",
      "args": ["@adyen/mcp-server"],
      "env": {
        "ADYEN_API_KEY": "your-api-key",
        "ADYEN_MERCHANT_ACCOUNT": "your-merchant-account",
        "ADYEN_ENVIRONMENT": "test"
      }
    }
  }
}
```

## Use Cases
1. Enterprise payment orchestration agent managing multi-region transactions
2. POS terminal fleet monitoring and configuration automation
3. Payout automation for marketplace seller disbursements

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Fraud detection | financial-fraud-detection | Real-time ML scoring before payment authorization |
| Risk analytics | nemotron-reasoning | Analyze payment patterns for anomaly detection |
| Commerce workflows | retail-agentic-commerce | End-to-end checkout with European payment methods |

## Prerequisites
- Adyen merchant account
- `ADYEN_API_KEY` — API key from Customer Area
- `ADYEN_MERCHANT_ACCOUNT` — Merchant account identifier
- Node.js 18+

## Security Notes
- Use test environment credentials during development
- Implement HMAC signature verification for webhooks
- Restrict API key roles to minimum required permissions
- Enable IP allowlisting in Adyen Customer Area

## References
- https://github.com/Adyen/adyen-mcp
- https://docs.adyen.com/development-resources/mcp-server
- https://docs.adyen.com/api-explorer/
