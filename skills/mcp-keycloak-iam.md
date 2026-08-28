# Keycloak MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | keycloak-model-context-protocol |
| **Category** | Identity/IAM |
| **Official** | Community |
| **Source** | https://github.com/ChristophEnglisch/keycloak-model-context-protocol |
| **Transport** | stdio |
| **Install** | `pip install keycloak-mcp` |

## Description
Community Keycloak MCP server for managing self-hosted identity and access management. Enables AI-assisted administration of realms, users, groups, roles, and client applications for organizations running Keycloak as their IAM solution with full control over authentication flows.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_users` | List users in a realm |
| `create_user` | Create a new user in a realm |
| `list_realms` | List all configured realms |
| `list_groups` | List groups within a realm |
| `assign_role` | Assign a role to a user |
| `list_clients` | List registered client applications |
| `get_user_sessions` | Get active sessions for a user |

## Configuration
```json
{
  "mcpServers": {
    "keycloak": {
      "command": "python",
      "args": ["-m", "keycloak_mcp"],
      "env": {
        "KEYCLOAK_URL": "http://localhost:8080",
        "KEYCLOAK_ADMIN": "admin",
        "KEYCLOAK_PASSWORD": "admin-password"
      }
    }
  }
}
```

## Use Cases
1. Manage self-hosted authentication for air-gapped AI deployment environments
2. Provision users and assign roles for accessing inference service endpoints
3. Administer multi-tenant realms for isolated model serving environments

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Self-hosted auth | aiq-deploy | Provide self-hosted IAM for deployed AIQ services in private infrastructure |

## Prerequisites
- Keycloak server running (17.0+ recommended)
- `KEYCLOAK_URL` — Keycloak server base URL
- `KEYCLOAK_ADMIN` — Admin username
- `KEYCLOAK_PASSWORD` — Admin password

## Security Notes
- Never use default admin credentials in production; create dedicated service accounts
- Enable HTTPS and restrict admin console access to trusted networks

## References
- https://github.com/ChristophEnglisch/keycloak-model-context-protocol
- https://www.keycloak.org/docs/latest/server_admin/
