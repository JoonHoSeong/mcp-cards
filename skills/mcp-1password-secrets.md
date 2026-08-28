# 1Password MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | 1Password-MCP |
| **Category** | Secrets |
| **Official** | Community |
| **Source** | https://github.com/CakeRepository/1Password-MCP |
| **Transport** | stdio |
| **Install** | `npx 1password-mcp` |

## Description
Community 1Password MCP server for managing secrets, credentials, and sensitive items across vaults. Enables AI-assisted secret lifecycle management including retrieval, creation, sharing, and OTP generation for secure credential workflows in development and deployment pipelines.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_vaults` | List all accessible vaults |
| `list_items` | List items within a vault |
| `get_item` | Retrieve a specific secret item |
| `create_item` | Create a new secret item |
| `update_item` | Update an existing item's fields |
| `get_otp` | Generate one-time password for an item |
| `share_item` | Create a sharing link for an item |

## Configuration
```json
{
  "mcpServers": {
    "1password": {
      "command": "npx",
      "args": ["1password-mcp"],
      "env": {
        "OP_SERVICE_ACCOUNT_TOKEN": "your-service-account-token"
      }
    }
  }
}
```

## Use Cases
1. Inject training secrets (API keys, model registry creds) into Kubernetes jobs
2. Retrieve database credentials for inference service configuration
3. Manage and rotate secrets used across GPU cluster deployment pipelines

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Inject training secrets | tao-run-on-kubernetes | Securely inject credentials into TAO training jobs on Kubernetes |

## Prerequisites
- 1Password account with service account configured
- `OP_SERVICE_ACCOUNT_TOKEN` — Service account token with vault access

## Security Notes
- Service account tokens grant broad vault access; scope to minimal required vaults
- Audit secret access logs regularly; avoid caching retrieved secrets in plaintext

## References
- https://github.com/CakeRepository/1Password-MCP
- https://developer.1password.com/docs/service-accounts/
