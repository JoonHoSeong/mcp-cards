# Atlassian MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Atlassian MCP |
| **Category** | Project Management |
| **Official** | Official |
| **Source** | https://github.com/sooperset/mcp-atlassian |
| **Transport** | stdio |
| **Install** | `npx -y mcp-atlassian` |

## Description
Atlassian MCP server provides unified access to both Jira and Confluence, enabling AI agents to manage issues, search project boards, create and edit documentation pages, and navigate knowledge spaces. It bridges project tracking with documentation workflows for teams using the Atlassian ecosystem.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_jira` | Search Jira issues using JQL queries |
| `create_issue` | Create a Jira issue with type, priority, and custom fields |
| `update_issue` | Update issue fields, status transitions, and comments |
| `list_projects` | List all Jira projects with lead and category info |
| `search_confluence` | Search Confluence pages by content or title |
| `get_page` | Retrieve a Confluence page with full content |
| `create_page` | Create a new Confluence page in a space |
| `list_spaces` | List Confluence spaces with permissions info |

## Configuration
```json
{
  "mcpServers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "mcp-atlassian"],
      "env": {
        "JIRA_URL": "https://your-org.atlassian.net",
        "JIRA_TOKEN": "<your-api-token>",
        "CONFLUENCE_URL": "https://your-org.atlassian.net/wiki",
        "JIRA_EMAIL": "your-email@company.com"
      }
    }
  }
}
```

## Use Cases
1. Auto-create Jira issues from code review findings or CI failures  2. Search and update Confluence documentation during development  3. Synchronize sprint planning with automated progress tracking

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Jira integration | mcore-create-issue | Create and manage Megatron-Core tracking issues in Jira |
| Document GPU configs | jetson-init-target | Auto-generate Confluence pages for Jetson deployment configurations |
| Track deployment issues | aiq-deploy | Create Jira tickets for AIQ deployment failures with diagnostics |

## Prerequisites
- `JIRA_URL` — Atlassian instance base URL
- `JIRA_TOKEN` — API token from Atlassian account settings
- `CONFLUENCE_URL` — Confluence wiki base URL
- Email associated with the API token

## Security Notes
- API token grants access to all projects the user can see; use service accounts with minimal permissions
- JQL queries can expose sensitive issue data; audit search patterns
- Confluence pages may contain confidential information; respect space permissions

## References
- https://github.com/sooperset/mcp-atlassian
- https://developer.atlassian.com/cloud/jira/platform/rest/v3/
- https://developer.atlassian.com/cloud/confluence/rest/v2/
