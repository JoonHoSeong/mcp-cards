# Webflow CMS & Site MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Webflow CMS & Site MCP Server |
| **Category** | CMS/No-Code |
| **Official** | ⭐ Official (Webflow) |
| **Source** | https://github.com/webflow/mcp-webflow |
| **Transport** | stdio |
| **Install** | `npx -y @webflow/mcp-server` |

## Tools & Capabilities
- `list_cms_items` — Query CMS collection items, custom field values, and draft statuses
- `create_cms_item` — Insert new content item into Webflow CMS collection with slug and metadata
- `publish_site` — Trigger live site publish across custom domains and staging subdomains
- `get_site_structure` — Inspect pages, collections, schemas, and asset links

## Client Configuration
```json
{
  "mcpServers": {
    "webflow": {
      "command": "npx",
      "args": [
        "-y",
        "@webflow/mcp-server"
      ],
      "env": {
        "WEBFLOW_API_TOKEN": "YOUR_WEBFLOW_TOKEN",
        "WEBFLOW_SITE_ID": "YOUR_SITE_ID"
      }
    }
  }
}
```

## Security & Best Practices
- Scoped Webflow API token. Site publish operations require explicit user approval.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
