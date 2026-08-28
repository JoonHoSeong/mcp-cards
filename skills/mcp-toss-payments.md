# Toss Payments MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Toss Payments MCP Server |
| **Category** | Fintech/Korea |
| **Official** | ⭐ Official (Toss Payments) |
| **Source** | https://github.com/tosspayments/mcp-tosspayments |
| **Transport** | stdio |
| **Install** | `npx -y @tosspayments/mcp-server` |

## Tools & Capabilities
- `confirm_payment` — Approve and finalize payment transaction using paymentKey and orderId
- `get_payment_by_order_id` — Query transaction details, receipt URLs, and card authorization codes
- `cancel_payment` — Issue partial or full refund with structured cancellation reason
- `issue_virtual_account` — Generate dedicated virtual bank account number for wire transfers
- `query_settlement_history` — Retrieve merchant payout and settlement reports

## Client Configuration
```json
{
  "mcpServers": {
    "toss-payments": {
      "command": "npx",
      "args": [
        "-y",
        "@tosspayments/mcp-server"
      ],
      "env": {
        "TOSS_SECRET_KEY": "test_sk_..."
      }
    }
  }
}
```

## Security & Best Practices
- Test in Sandbox before switching to live credentials. Require explicit confirmation for refunds.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
