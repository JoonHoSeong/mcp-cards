# WhatsApp Business Cloud MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | WhatsApp Business Cloud MCP Server |
| **Category** | Communication/Messaging |
| **Official** | ⭐ Official / Meta Graph |
| **Source** | https://github.com/meta/mcp-whatsapp-business |
| **Transport** | stdio |
| **Install** | `npx -y @meta/mcp-whatsapp` |

## Tools & Capabilities
- `send_template_message` — Dispatch pre-approved HSM notification template to customer
- `send_text_message` — Send interactive customer support reply within 24hr service window
- `send_media` — Upload and deliver PDF invoices, receipts, and product images
- `get_message_status` — Track message delivery, sent, and read receipt timestamps
- `manage_business_profile` — Inspect business hours, contact info, and catalog links

## Client Configuration
```json
{
  "mcpServers": {
    "whatsapp": {
      "command": "npx",
      "args": [
        "-y",
        "@meta/mcp-whatsapp"
      ],
      "env": {
        "WHATSAPP_TOKEN": "YOUR_META_GRAPH_ACCESS_TOKEN",
        "PHONE_NUMBER_ID": "YOUR_PHONE_NUMBER_ID"
      }
    }
  }
}
```

## Security & Best Practices
- Comply with Meta 24-hour customer care window and opt-in messaging policies.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
