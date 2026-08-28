# Strapi Headless CMS MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Strapi Headless CMS MCP Server |
| **Category** | CMS/Backend |
| **Official** | ⭐ Official (Strapi) |
| **Source** | https://github.com/strapi/mcp-strapi |
| **Transport** | stdio |
| **Install** | `npx -y @strapi/mcp-server` |

## Tools & Capabilities
- `find_entries` — Query content entries with populate, deep filtering, and pagination
- `create_entry` — Create new draft or published content entry in designated content type
- `update_entry` — Modify existing content entry fields and publish state
- `list_content_types` — Inspect available schemas, components, and media fields
- `upload_media` — Upload images or documents to Strapi Media Library

## Client Configuration
```json
{
  "mcpServers": {
    "strapi": {
      "command": "npx",
      "args": [
        "-y",
        "@strapi/mcp-server"
      ],
      "env": {
        "STRAPI_URL": "http://localhost:1337",
        "STRAPI_API_TOKEN": "YOUR_STRAPI_API_TOKEN"
      }
    }
  }
}
```

## Security & Best Practices
- Scoped Custom API tokens. Enforce draft/publish workflow rules.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
