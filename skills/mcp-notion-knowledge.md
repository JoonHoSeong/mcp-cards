# Notion MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Notion MCP |
| **Category** | Knowledge Management |
| **Official** | Yes (hosted remote) |
| **Source** | https://mcp.notion.com/mcp |
| **Transport** | Remote (HTTP + SSE, OAuth) |
| **Install** | Remote server — no local install required |

## Description
The Notion MCP server provides AI agents with access to Notion workspaces for reading and writing pages, querying databases, and searching across knowledge bases. It enables agents to leverage organizational knowledge, create documentation, and maintain structured data within Notion.

## Key Tools
| Tool | Description |
|------|-------------|
| `search` | Search across workspace pages and databases |
| `get_page` | Retrieve a page and its content |
| `create_page` | Create a new page in a workspace |
| `update_page` | Update existing page properties or content |
| `query_database` | Query a database with filters and sorts |
| `create_database_item` | Add a new item to a database |
| `append_block` | Append content blocks to a page |
| `list_users` | List workspace users |

## Configuration
```json
{
  "mcpServers": {
    "notion": {
      "url": "https://mcp.notion.com/mcp"
    }
  }
}
```

## Use Cases
1. AI-powered knowledge retrieval from organizational wikis
2. Automated documentation creation and maintenance
3. Structured data management via database queries and updates

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Document skills | nvidia-skill-finder | Index Notion docs as discoverable AI skills for agent orchestration |

## Prerequisites
- Notion account with OAuth authorization for MCP access

## Security Notes
- OAuth scoping limits which pages and databases the agent can access
- Workspace data may contain sensitive internal information; apply least-privilege access

## References
- https://mcp.notion.com/mcp
- https://developers.notion.com/reference
