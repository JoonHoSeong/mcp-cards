# Wise (TransferWise) International Payments MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Wise (TransferWise) International Payments MCP Server |
| **Category** | Fintech/Payments |
| **Official** | ⭐ Official (Wise) |
| **Source** | https://github.com/wise/mcp-wise |
| **Transport** | stdio |
| **Install** | `npx -y @wise/mcp-server` |

## Tools & Capabilities
- `get_exchange_rate` — Fetch real-time mid-market currency exchange rates and transfer fees
- `create_quote` — Generate guaranteed international transfer quote with delivery estimates
- `list_recipients` — Query saved bank recipient profiles across global currencies
- `create_transfer` — Initiate borderless money transfer with quote ID reference
- `get_transfer_status` — Track live payment status and SWIFT/SEPA settlement milestones

## Client Configuration
```json
{
  "mcpServers": {
    "wise": {
      "command": "npx",
      "args": [
        "-y",
        "@wise/mcp-server"
      ],
      "env": {
        "WISE_API_TOKEN": "YOUR_WISE_API_TOKEN",
        "WISE_PROFILE_ID": "YOUR_PROFILE_ID"
      }
    }
  }
}
```

## Security & Best Practices
- Require multi-factor authorization before committing fund transfer orders.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
