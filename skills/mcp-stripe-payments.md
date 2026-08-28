# Stripe MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Stripe MCP Server |
| **Category** | Payments/Billing |
| **Official** | ⭐ Official (Stripe) |
| **Source** | https://github.com/stripe/agent-toolkit |
| **Transport** | stdio |
| **Install** | `npx -y @stripe/mcp --tools=all` |

## Description
The Stripe MCP Server is Stripe's official agent toolkit that enables AI assistants to interact with the Stripe payments platform. It provides tools for creating payment links, managing invoices, querying customers, handling subscriptions, and processing refunds — all through conversational interfaces that translate natural language into precise Stripe API operations.

This server is part of Stripe's broader agent toolkit strategy, designed for businesses that want to integrate payment operations into AI-powered workflows. Whether it's a support agent issuing refunds, a sales tool generating invoices, or an analytics bot summarizing revenue, the MCP server provides safe, auditable access to Stripe's full billing infrastructure.

The `--tools=all` flag enables the complete toolset, but operators can restrict available tools to specific subsets (e.g., read-only for analytics use cases). This granular control is critical for payment systems where different roles require different permission levels.

## Key Tools
| Tool | Description |
|------|-------------|
| create_payment_link | Generate a shareable payment link for products or custom amounts |
| create_invoice | Create and optionally send an invoice to a customer |
| list_customers | List customers with optional filtering |
| get_customer | Retrieve detailed information about a specific customer |
| list_subscriptions | List active subscriptions with status filtering |
| get_balance | Get the current Stripe account balance |
| list_payouts | List recent payouts to connected bank accounts |
| create_refund | Issue a full or partial refund for a charge |
| list_charges | List recent charges with filtering options |
| list_products | List products in the catalog |
| create_product | Create a new product in the Stripe catalog |
| create_price | Create a pricing tier for a product |

## Configuration
```json
{
  "mcpServers": {
    "stripe": {
      "command": "npx",
      "args": ["-y", "@stripe/mcp", "--tools=all"],
      "env": {
        "STRIPE_SECRET_KEY": "<your-stripe-secret-key>"
      }
    }
  }
}
```

## Use Cases
1. Generate invoices via chat — create and send invoices to customers through natural language
2. Lookup customer payment status — check subscription status, outstanding balances, and payment history
3. Issue refunds — process full or partial refunds with proper audit trails
4. Create subscription products — set up new products and pricing tiers conversationally
5. Revenue analytics — query charges, payouts, and balance for financial reporting

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Monetize AI agents | aiq-deploy | Add billing and payment collection to deployed AI agent services |
| Billing for RAG service | rag-blueprint | Charge customers for RAG-based retrieval API usage |
| Charge for custom model API | nemotron-customize | Create pricing tiers for fine-tuned Nemotron model endpoints |

## Prerequisites
- Stripe account (test or live mode)
- `STRIPE_SECRET_KEY` environment variable configured
- Node.js runtime for npx execution

## Security Notes
- Use restricted API keys with only the permissions needed for the specific use case
- Never log or display Stripe secret keys in outputs or conversation history
- Gate destructive operations (refunds, deletions) behind explicit user confirmation
- Use Stripe's test mode keys (`sk_test_*`) during development and testing
- Monitor webhook events for anomalous activity triggered by agent operations
- Consider using Stripe's built-in fraud detection alongside agent-initiated payments

## References
- https://github.com/stripe/agent-toolkit
- https://docs.stripe.com/api
- https://docs.stripe.com/keys
