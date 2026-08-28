# HashiCorp Vault MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | HashiCorp Vault MCP Server |
| **Category** | Security/Secrets Management |
| **Official** | ⭐ Official (HashiCorp) |
| **Source** | https://github.com/hashicorp/vault-mcp-server |
| **Transport** | stdio + Streamable HTTP |
| **Install** | `vault-mcp-server` (binary) |

## Description
MCP server for HashiCorp Vault that enables AI agents to securely manage secrets, credentials, and PKI certificates. Provides programmatic access to Vault's secrets engines, policies, and dynamic credential generation through the Model Context Protocol.

## Key Tools
| Tool | Description |
|------|-------------|
| `read_secret` | Read a secret from a specified path in Vault |
| `write_secret` | Write or update a secret at a specified path |
| `list_secrets` | List all secrets at a given path |
| `list_mounts` | List all mounted secrets engines |
| `enable_secret_engine` | Enable a new secrets engine at a path |
| `read_policy` | Read a Vault ACL policy |
| `list_policies` | List all configured policies |
| `seal_status` | Check the seal status of the Vault server |
| `generate_credentials` | Generate dynamic credentials from a secrets engine |

## Configuration
```json
{
  "mcpServers": {
    "vault": {
      "command": "vault-mcp-server",
      "args": ["--address", "https://vault.example.com:8200"],
      "env": {
        "VAULT_ADDR": "https://vault.example.com:8200",
        "VAULT_TOKEN": "hvs.your-token-here"
      }
    }
  }
}
```

## Use Cases
1. Manage application secrets directly from the development environment
2. Rotate credentials automatically during CI/CD pipelines
3. Provision dynamic database credentials for ephemeral workloads
4. Audit secret access patterns and policy compliance
5. Manage PKI certificates for service mesh and mTLS

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Inject training secrets | tao-run-on-kubernetes | Securely provide API keys and credentials to TAO training jobs on Kubernetes |
| Distribute cluster credentials | nemo-mbridge-multi-node-slurm | Manage SSH keys and tokens across multi-node Slurm clusters |
| Secure DPU credentials | doca-setup | Store and retrieve DPU authentication credentials securely |

## Prerequisites
- HashiCorp Vault server (v1.15+) running and accessible
- `VAULT_ADDR` environment variable pointing to Vault server
- `VAULT_TOKEN` or AppRole credentials for authentication
- `vault-mcp-server` binary installed and in PATH

## Security Notes
- Use AppRole or token authentication with minimal policies — never expose the root token
- Apply least-privilege policies scoped to specific paths and operations
- Enable audit logging to track all secret access through the MCP server
- Rotate tokens regularly and use short TTLs for dynamic credentials
- Never store `VAULT_TOKEN` in plaintext configuration files in shared repositories

## References
- https://developer.hashicorp.com/vault/docs/mcp-server/deploy
- https://github.com/hashicorp/vault-mcp-server
- https://developer.hashicorp.com/vault/docs/concepts/policies
- https://developer.hashicorp.com/vault/docs/auth/approle
