# Cloudflare MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Cloudflare MCP |
| **Category** | Cloud/Edge |
| **Official** | Yes (Cloudflare) |
| **Source** | https://github.com/cloudflare/mcp-server-cloudflare |
| **Transport** | stdio |
| **Install** | `npx @cloudflare/mcp-server-cloudflare` |

## Description
The Cloudflare MCP Server enables AI agents to manage Cloudflare Workers, KV namespaces, R2 object storage, D1 databases, and DNS zones. It provides a comprehensive interface for deploying edge compute workloads, managing serverless storage, and configuring network services across Cloudflare's global edge network.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_workers` | List all deployed Workers scripts |
| `deploy_worker` | Deploy or update a Worker script |
| `list_kv_namespaces` | List KV storage namespaces |
| `kv_get` | Read a value from KV storage |
| `kv_put` | Write a value to KV storage |
| `list_r2_buckets` | List R2 object storage buckets |
| `r2_upload` | Upload an object to R2 storage |
| `query_d1` | Execute SQL queries against D1 databases |
| `manage_dns` | Create, update, or delete DNS records |
| `list_zones` | List all DNS zones in the account |

## Configuration
```json
{
  "mcpServers": {
    "cloudflare": {
      "command": "npx",
      "args": ["@cloudflare/mcp-server-cloudflare"],
      "env": {
        "CLOUDFLARE_API_TOKEN": "your-api-token",
        "CLOUDFLARE_ACCOUNT_ID": "your-account-id"
      }
    }
  }
}
```

## Use Cases
1. Deploy edge caching layers for AI inference API responses globally
2. Store and retrieve RAG document embeddings in KV for low-latency access
3. Manage DNS and routing for distributed AI service endpoints

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Edge caching | rag-blueprint | Cache RAG query results at the edge for faster retrieval |
| Response delivery | rag-blueprint | Serve precomputed embeddings from R2/KV near users |
| API gateway | rag-blueprint | Route inference requests through Workers to NVIDIA endpoints |

## Prerequisites
- Cloudflare account with Workers enabled
- `CLOUDFLARE_API_TOKEN` with appropriate permissions
- Account ID for targeting specific resources

## Security Notes
- Use scoped API tokens with minimum required permissions
- Never use Global API Key; prefer token-based authentication
- Enable Workers analytics for monitoring unusual activity
- Restrict KV/R2 access to specific Workers only

## References
- https://github.com/cloudflare/mcp-server-cloudflare
- https://developers.cloudflare.com/workers/
- https://developers.cloudflare.com/kv/
