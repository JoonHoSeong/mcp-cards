# PayPal Agent Toolkit MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | paypal-agent-toolkit |
| **Category** | Payments/Billing |
| **Official** | Yes (PayPal) |
| **Source** | https://github.com/paypal/agent-toolkit |
| **Transport** | stdio |
| **Install** | `npm install @paypal/agent-toolkit` |

## Description
PayPal's official Agent Toolkit provides MCP server capabilities for integrating PayPal payment processing into AI agents. It enables order creation, payment capture, invoicing, refunds, and dispute management through PayPal's REST APIs, supporting both sandbox and production environments.

## Key Tools
| Tool | Description |
|------|-------------|
| `create_order` | Create a new PayPal order with items and amounts |
| `capture_payment` | Capture an authorized payment |
| `list_transactions` | List recent transactions with filters |
| `get_transaction` | Get details of a specific transaction |
| `create_invoice` | Create a draft invoice |
| `send_invoice` | Send an invoice to a recipient |
| `issue_refund` | Issue a full or partial refund |
| `list_disputes` | List open payment disputes |

## Configuration
```json
{
  "mcpServers": {
    "paypal": {
      "command": "npx",
      "args": ["@paypal/agent-toolkit", "mcp"],
      "env": {
        "PAYPAL_CLIENT_ID": "your-client-id",
        "PAYPAL_CLIENT_SECRET": "your-client-secret",
        "PAYPAL_ENVIRONMENT": "sandbox"
      }
    }
  }
}
```

## Use Cases
1. AI agent that processes customer orders and captures payments automatically
2. Automated invoice generation and sending for freelancer/contractor workflows
3. Dispute monitoring and refund processing agent for customer support

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Monetize AI agents | aiq-deploy | Deploy agents that charge via PayPal for inference or services |
| Fraud detection | financial-fraud-detection | Screen transactions before capture |
| Customer support | nemotron-reasoning | Intelligent dispute resolution recommendations |

## Prerequisites
- PayPal Developer account with REST API credentials
- `PAYPAL_CLIENT_ID` — OAuth client ID
- `PAYPAL_CLIENT_SECRET` — OAuth client secret
- Node.js 18+

## Security Notes
- Use sandbox environment for development and testing
- Never expose client secrets in client-side code
- Implement webhook signature verification for payment notifications
- Store credentials in secure vault, not environment files

## References
- https://github.com/paypal/agent-toolkit
- https://developer.paypal.com/docs/api/overview/
- https://developer.paypal.com/docs/checkout/
