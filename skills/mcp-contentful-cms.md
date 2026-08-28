# Contentful Headless CMS MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Contentful Headless CMS MCP Server |
| **Category** | CMS/Enterprise |
| **Official** | ⭐ Official (Contentful) |
| **Source** | https://github.com/contentful/mcp-contentful |
| **Transport** | stdio |
| **Install** | `npx -y @contentful/mcp-server` |

## Tools & Capabilities
- `get_entries` — Query content entries across spaces and environments with localization
- `create_entry` — Create new entry with localized field values
- `publish_entry` — Publish entry revision to live production Content Delivery API
- `get_content_types` — Inspect content models, validation constraints, and field types
- `manage_assets` — Upload, process, and publish media assets to Contentful CDN

## Client Configuration
```json
{
  "mcpServers": {
    "contentful": {
      "command": "npx",
      "args": [
        "-y",
        "@contentful/mcp-server"
      ],
      "env": {
        "CONTENTFUL_SPACE_ID": "YOUR_SPACE_ID",
        "CONTENTFUL_ENVIRONMENT": "master",
        "CONTENTFUL_CMA_TOKEN": "CFPAT-..."
      }
    }
  }
}
```

## Security & Best Practices
- Content Management API (CMA) token access. Separate staging and master environment write operations.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
