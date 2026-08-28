# Plaid Open Banking MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Plaid Open Banking MCP Server |
| **Category** | Fintech/Banking |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/plaid/mcp-plaid |
| **Transport** | stdio |
| **Install** | `npx -y @plaid/mcp-server` |

## Tools & Capabilities
- `get_accounts` — Query connected bank account balances, types, and account masks
- `get_transactions` — Fetch historical categorized bank transactions with date range filters
- `create_link_token` — Generate Plaid Link token for frontend bank connection widgets
- `get_identity` — Verify account owner identity, phone, and address data
- `get_recurring_transactions` — Detect recurring subscriptions, payroll deposits, and bills

## Client Configuration
```json
{
  "mcpServers": {
    "plaid": {
      "command": "npx",
      "args": [
        "-y",
        "@plaid/mcp-server"
      ],
      "env": {
        "PLAID_CLIENT_ID": "YOUR_PLAID_CLIENT_ID",
        "PLAID_SECRET": "YOUR_PLAID_SECRET",
        "PLAID_ENV": "sandbox"
      }
    }
  }
}
```

## Security & Best Practices
- Strict financial compliance (GLBA/SOC2). Never log unmasked bank account or routing numbers.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
