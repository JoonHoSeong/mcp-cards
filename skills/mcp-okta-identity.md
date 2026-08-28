# Okta MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | okta-mcp-server |
| **Category** | Identity |
| **Official** | Community |
| **Source** | https://github.com/fctr-id/okta-mcp-server |
| **Transport** | stdio |
| **Install** | `npx okta-mcp-server` |

## Description
Community Okta MCP server for managing enterprise identity, users, groups, and MFA policies. Enables AI-assisted identity administration including user lifecycle management, group membership queries, sign-in log analysis, and privileged access monitoring across Okta-managed organizations.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_users` | List users with search and filter capabilities |
| `get_user` | Get detailed user profile information |
| `list_groups` | List groups and their memberships |
| `get_mfa_status` | Check MFA enrollment status for a user |
| `list_applications` | List configured applications |
| `get_signin_logs` | Retrieve user sign-in event logs |
| `list_privileged_users` | List users with admin privileges |

## Configuration
```json
{
  "mcpServers": {
    "okta": {
      "command": "npx",
      "args": ["okta-mcp-server"],
      "env": {
        "OKTA_DOMAIN": "your-org.okta.com",
        "OKTA_API_TOKEN": "your-api-token"
      }
    }
  }
}
```

## Use Cases
1. Manage enterprise IAM policies for teams accessing DPU infrastructure
2. Audit privileged user access and MFA compliance across AI platform deployments
3. Query sign-in logs to investigate unauthorized access attempts to GPU clusters

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Enterprise IAM for DPU | doca-setup | Manage enterprise identity and access for DOCA DPU environments |

## Prerequisites
- Okta organization with API access enabled
- `OKTA_DOMAIN` — Your Okta organization domain (e.g., dev-123456.okta.com)
- `OKTA_API_TOKEN` — API token created in Okta admin console

## Security Notes
- API tokens inherit the permissions of the admin who created them; use least-privilege admins
- Token activity is logged; rotate tokens on personnel changes

## References
- https://github.com/fctr-id/okta-mcp-server
- https://developer.okta.com/docs/reference/api/
