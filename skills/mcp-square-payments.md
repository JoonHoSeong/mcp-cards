# Square Payments & POS MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Square Payments & POS MCP Server |
| **Category** | Fintech/Payments |
| **Official** | ⭐ Official (Square) |
| **Source** | https://github.com/square/mcp-square |
| **Transport** | stdio |
| **Install** | `npx -y @square/mcp-server` |

## Tools & Capabilities
- `list_payments` — Query processed transactions, payment methods, and receipt URLs
- `create_payment_link` — Generate hosted checkout payment links with line item totals
- `get_order_details` — Retrieve customer order status, fulfillment states, and tax breakdowns
- `list_customers` — Search customer profiles, cards on file, and loyalty balances
- `manage_inventory` — Check item stock counts across physical locations

## Client Configuration
```json
{
  "mcpServers": {
    "square": {
      "command": "npx",
      "args": [
        "-y",
        "@square/mcp-server"
      ],
      "env": {
        "SQUARE_ACCESS_TOKEN": "YOUR_SQUARE_ACCESS_TOKEN",
        "SQUARE_ENVIRONMENT": "sandbox"
      }
    }
  }
}
```

## Security & Best Practices
- Test thoroughly in sandbox mode before production. Require 2FA approval for refund actions.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
