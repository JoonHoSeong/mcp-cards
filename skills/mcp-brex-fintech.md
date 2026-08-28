# Brex Corporate Cards & Expenses MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Brex Corporate Cards & Expenses MCP Server |
| **Category** | Fintech/Corporate |
| **Official** | ⭐ Official (Brex) |
| **Source** | https://github.com/brex/mcp-brex |
| **Transport** | stdio |
| **Install** | `npx -y @brex/mcp-server` |

## Tools & Capabilities
- `list_expenses` — Query card transactions, receipt attachments, and accounting categories
- `get_card_limits` — Inspect corporate card spend limits, available credit, and owner details
- `upload_receipt` — Attach digital receipt images or PDF files to specific expense records
- `list_budgets` — Review department budget allocations, burn rates, and remaining allowances
- `manage_card_locks` — Freeze or unfreeze corporate cards in response to security alerts

## Client Configuration
```json
{
  "mcpServers": {
    "brex": {
      "command": "npx",
      "args": [
        "-y",
        "@brex/mcp-server"
      ],
      "env": {
        "BREX_API_TOKEN": "YOUR_BREX_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Enforce read-only auditing by default. Card lock and budget mutations require admin elevation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
