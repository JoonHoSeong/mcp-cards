# Chargebee MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | chargebee-agentkit |
| **Category** | Payments/Billing |
| **Official** | Yes (Chargebee) |
| **Source** | https://github.com/chargebee/agentkit |
| **Transport** | stdio |
| **Install** | `npm install @chargebee/agentkit` |

## Description
Chargebee's official AgentKit MCP server provides AI agents with full subscription lifecycle management capabilities. It handles plan management, customer billing, invoicing, revenue metrics, and subscription operations for SaaS businesses, supporting complex billing models including usage-based, tiered, and hybrid pricing.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_subscriptions` | List all subscriptions with status filters |
| `create_subscription` | Create a new subscription for a customer |
| `cancel_subscription` | Cancel or schedule cancellation |
| `list_invoices` | Retrieve invoices with date/status filters |
| `create_invoice` | Generate a one-time or recurring invoice |
| `list_customers` | List customers with search criteria |
| `get_revenue_metrics` | Fetch MRR, ARR, churn, and growth metrics |
| `manage_plans` | Create, update, or archive pricing plans |

## Configuration
```json
{
  "mcpServers": {
    "chargebee": {
      "command": "npx",
      "args": ["@chargebee/agentkit", "mcp"],
      "env": {
        "CHARGEBEE_SITE": "your-site-name",
        "CHARGEBEE_API_KEY": "your-api-key"
      }
    }
  }
}
```

## Use Cases
1. AI agent managing SaaS subscription upgrades, downgrades, and cancellations
2. Automated revenue reporting and churn analysis for finance teams
3. Customer billing support agent resolving invoice and payment issues

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Subscription-gated access | nemotron-customize | Gate fine-tuned model access behind subscription tiers |
| Revenue forecasting | portfolio-optimization | Predict MRR growth and churn patterns |
| Customer support | nemotron-reasoning | Intelligent billing issue resolution |

## Prerequisites
- Chargebee account with API access
- `CHARGEBEE_SITE` — Your Chargebee site name
- `CHARGEBEE_API_KEY` — Full-access or read-only API key
- Node.js 18+

## Security Notes
- Use read-only API keys for analytics-only agents
- Restrict API key permissions per use case
- Audit subscription changes via Chargebee webhooks
- Never expose API keys in client-side applications

## References
- https://github.com/chargebee/agentkit
- https://apidocs.chargebee.com/docs/api
- https://www.chargebee.com/docs/2.0/
