# Razorpay MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | razorpay-mcp-server |
| **Category** | Payments/Billing |
| **Official** | Yes (Razorpay) |
| **Source** | https://github.com/razorpay/razorpay-mcp-server |
| **Transport** | stdio |
| **Install** | `npm install @razorpay/mcp-server` |

## Description
Razorpay's official MCP server provides AI agents with payment processing capabilities tailored for India and Southeast Asia. Supporting UPI, credit/debit cards, netbanking, wallets, and EMI, it enables agents to manage payments, refunds, subscriptions, settlements, and revenue analytics across the region's diverse payment landscape.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_payments` | List payments with date and status filters |
| `get_payment` | Get detailed payment information |
| `create_payment_link` | Generate shareable payment links |
| `list_refunds` | List refunds with filters |
| `create_refund` | Issue full or partial refund |
| `list_subscriptions` | List recurring subscriptions |
| `get_settlement` | Get settlement details and timeline |
| `revenue_analytics` | Fetch revenue and payment method analytics |

## Configuration
```json
{
  "mcpServers": {
    "razorpay": {
      "command": "npx",
      "args": ["@razorpay/mcp-server"],
      "env": {
        "RAZORPAY_KEY_ID": "your-key-id",
        "RAZORPAY_KEY_SECRET": "your-key-secret"
      }
    }
  }
}
```

## Use Cases
1. E-commerce agent processing UPI and card payments for Indian merchants
2. Automated refund and settlement reconciliation for finance teams
3. Payment link generation agent for remote sales and collections

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Retail commerce | retail-agentic-commerce | End-to-end Indian e-commerce with local payment methods |
| Fraud detection | financial-fraud-detection | Screen UPI and card transactions for fraud |
| Analytics | nemotron-reasoning | Payment method and settlement trend analysis |

## Prerequisites
- Razorpay merchant account
- `RAZORPAY_KEY_ID` — API key ID from dashboard
- `RAZORPAY_KEY_SECRET` — API key secret
- Node.js 18+

## Security Notes
- Use test mode keys for development (prefix `rzp_test_`)
- Verify webhook signatures using Razorpay's signing algorithm
- Store key secret securely; never expose in frontend code
- Enable 2FA on Razorpay dashboard access

## References
- https://github.com/razorpay/razorpay-mcp-server
- https://razorpay.com/docs/api/
- https://razorpay.com/docs/payments/
