# WordPress REST API CMS MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | WordPress REST API CMS MCP Server |
| **Category** | CMS/Platform |
| **Official** | ⭐ Official / Verified |
| **Source** | https://github.com/wordpress/mcp-wordpress |
| **Transport** | stdio |
| **Install** | `npx -y @wordpress/mcp-server` |

## Tools & Capabilities
- `list_posts` — Query posts with category, tag, author, and status (draft/publish) filters
- `create_post` — Create new blog post or page with Gutenberg block content and featured media
- `update_post` — Modify existing post content, excerpts, categories, and SEO metadata
- `upload_media` — Upload images or attachments to WordPress Media Library

## Client Configuration
```json
{
  "mcpServers": {
    "wordpress": {
      "command": "npx",
      "args": [
        "-y",
        "@wordpress/mcp-server"
      ],
      "env": {
        "WP_URL": "https://your-site.com",
        "WP_APPLICATION_USERNAME": "admin",
        "WP_APPLICATION_PASSWORD": "xxxx xxxx xxxx xxxx"
      }
    }
  }
}
```

## Security & Best Practices
- WordPress Application Passwords authentication. Enforce draft state for automated edits.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
