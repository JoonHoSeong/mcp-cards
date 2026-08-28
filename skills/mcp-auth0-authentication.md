# Auth0 MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Auth0 MCP Server |
| **Category** | Security/Authentication |
| **Official** | ⭐ Official (Auth0/Okta) |
| **Source** | https://github.com/auth0/auth0-mcp-server |
| **Transport** | stdio |
| **Install** | `npx -y @auth0/mcp` |

## Description
Official Auth0 MCP server that enables AI agents to manage authentication and authorization configurations. Provides access to Auth0's Management API for managing applications, users, connections, roles, and organizations directly from the development environment.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_applications` | List all registered applications in the tenant |
| `get_application` | Get detailed configuration of a specific application |
| `list_users` | List and search users in the Auth0 tenant |
| `get_user` | Get detailed profile and metadata for a user |
| `create_user` | Create a new user in a database connection |
| `list_connections` | List identity provider connections (social, enterprise) |
| `get_logs` | Retrieve authentication and management audit logs |
| `list_roles` | List all roles defined in the tenant |
| `assign_role` | Assign a role to a user |
| `list_organizations` | List organizations (B2B multi-tenancy) |
| `get_branding` | Get tenant branding configuration |

## Configuration
```json
{
  "mcpServers": {
    "auth0": {
      "command": "npx",
      "args": ["-y", "@auth0/mcp"],
      "env": {
        "AUTH0_DOMAIN": "your-tenant.auth0.com",
        "AUTH0_CLIENT_ID": "your-m2m-client-id",
        "AUTH0_CLIENT_SECRET": "your-m2m-client-secret"
      }
    }
  }
}
```

## Use Cases
1. Manage authentication users and applications from the IDE
2. Troubleshoot login issues by reviewing authentication logs
3. Configure SSO connections for enterprise customers
4. Audit authentication logs for security compliance
5. Manage role-based access control assignments

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Secure AI-Q endpoints | aiq-deploy | Add authentication to AI-Q deployment API endpoints |
| Authenticate RAG API users | rag-blueprint | Protect RAG pipeline APIs with Auth0 user authentication |
| Gate custom model access | nemotron-customize | Control access to fine-tuned Nemotron model endpoints |

## Prerequisites
- Auth0 tenant (free tier or above)
- Machine-to-Machine (M2M) application configured in Auth0
- Management API authorized for the M2M application
- Required scopes granted to the M2M application (e.g., `read:users`, `read:logs`)

## Security Notes
- Use a dedicated M2M application with minimal Management API scopes
- Never expose the client secret in client-side code or shared repositories
- Grant only read scopes unless write operations are explicitly needed
- Rotate client secrets periodically
- Use separate tenants for development and production
- Audit log access provides visibility into all authentication events

## References
- https://github.com/auth0/auth0-mcp-server
- https://auth0.com/docs/api/management/v2
- https://auth0.com/docs/get-started/auth0-overview/create-applications/machine-to-machine-apps
- https://auth0.com/docs/secure/tokens/access-tokens/management-api-access-tokens
