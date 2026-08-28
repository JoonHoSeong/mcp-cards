# Clerk MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | mcp-tools |
| **Category** | Authentication |
| **Official** | Yes |
| **Source** | https://github.com/clerk/mcp-tools |
| **Transport** | stdio |
| **Install** | `npx @clerk/mcp-tools` |

## Description
Official Clerk MCP server for managing user authentication, session verification, and organization membership. Enables AI-assisted identity management workflows including user provisioning, session validation, and multi-tenant organization administration for applications using Clerk authentication.

## Key Tools
| Tool | Description |
|------|-------------|
| `verify_session` | Verify an active user session token |
| `get_user` | Get user profile and metadata |
| `list_users` | List users with filtering options |
| `create_user` | Create a new user account |
| `list_organizations` | List organizations in the instance |
| `get_org_membership` | Get organization membership details |

## Configuration
```json
{
  "mcpServers": {
    "clerk": {
      "command": "npx",
      "args": ["@clerk/mcp-tools"],
      "env": {
        "CLERK_SECRET_KEY": "sk_live_your-secret-key"
      }
    }
  }
}
```

## Use Cases
1. Secure AI inference API endpoints with session-based authentication
2. Manage user provisioning and organization access for AI platform deployments
3. Verify JWT sessions before granting access to model serving endpoints

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Secure AI endpoints | aiq-deploy | Add Clerk authentication to deployed AIQ inference endpoints |

## Prerequisites
- Clerk application with API keys configured
- `CLERK_SECRET_KEY` — Backend API secret key from Clerk dashboard

## Security Notes
- Never expose the secret key in client-side code; use only in server environments
- Implement webhook signature verification for Clerk event callbacks

## References
- https://github.com/clerk/mcp-tools
- https://clerk.com/docs/reference/backend-api
