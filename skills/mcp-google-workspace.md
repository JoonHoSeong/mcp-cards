# Google Workspace MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Google Workspace MCP Server |
| **Category** | Productivity/Office |
| **Official** | 커뮤니티 검증 |
| **Source** | https://github.com/googleworkspace/mcp-google-workspace |
| **Transport** | stdio |
| **Install** | `npx -y @google/mcp-workspace` |

## Tools & Capabilities
- `read_sheet` — Read spreadsheet cell ranges, headers, and computed values
- `append_sheet_row` — Append row data to specific Google Sheets tab
- `search_docs` — Search and extract text content from Google Docs files
- `list_calendar_events` — Query Google Calendar events within date range
- `send_email_draft` — Create or send email drafts via Gmail API

## Client Configuration
```json
{
  "mcpServers": {
    "google-workspace": {
      "command": "npx",
      "args": [
        "-y",
        "@google/mcp-workspace"
      ],
      "env": {
        "GOOGLE_WORKSPACE_CREDENTIALS": "/path/to/oauth-credentials.json"
      }
    }
  }
}
```

## Security & Best Practices
- Requires explicit OAuth2 scope consent. Send email operations require mandatory user confirmation.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
