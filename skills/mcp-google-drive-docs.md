# Google Drive MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Google Drive MCP |
| **Category** | Cloud Storage |
| **Official** | Reference Implementation |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/gdrive |
| **Transport** | stdio |
| **Install** | `npx @modelcontextprotocol/server-gdrive` |

## Description
The Google Drive MCP server provides AI agents with access to Google Drive files and folders. It enables searching, reading, and creating files, making it useful for sourcing training documents, managing shared assets, and integrating Drive content into AI workflows.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_files` | Search for files by name or content |
| `get_file` | Get file metadata and details |
| `list_files` | List files in a folder |
| `read_file_content` | Read the text content of a file |
| `create_file` | Create a new file in Drive |
| `share_file` | Share a file with users or groups |

## Configuration
```json
{
  "mcpServers": {
    "gdrive": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-gdrive"],
      "env": {
        "GOOGLE_CLIENT_ID": "${GOOGLE_CLIENT_ID}",
        "GOOGLE_CLIENT_SECRET": "${GOOGLE_CLIENT_SECRET}",
        "GOOGLE_REDIRECT_URI": "http://localhost:3000/oauth/callback"
      }
    }
  }
}
```

## Use Cases
1. Sourcing training documents and datasets from shared drives
2. Automated file organization and content extraction
3. Knowledge base construction from Google Docs and Sheets

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Source training docs | data-designer | Pull documents from Drive as source material for synthetic data generation |

## Prerequisites
- Google OAuth credentials (client ID, client secret) with Drive API scopes

## Security Notes
- OAuth tokens grant access to user's Drive files; scope to minimum required permissions
- Downloaded file content may contain sensitive business data

## References
- https://github.com/modelcontextprotocol/servers/tree/main/src/gdrive
- https://developers.google.com/drive/api
