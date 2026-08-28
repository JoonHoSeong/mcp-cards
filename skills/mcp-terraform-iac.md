# Terraform MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Terraform MCP Server |
| **Category** | Infrastructure/IaC |
| **Official** | ⭐ Official (HashiCorp) |
| **Source** | https://github.com/hashicorp/terraform-mcp-server |
| **Transport** | stdio / Streamable HTTP |
| **Install** | `npx -y @hashicorp/terraform-mcp-server` |

## Description
The Terraform MCP Server by HashiCorp provides AI assistants with direct access to Terraform provider documentation, module schemas, and resource definitions. It enables agents to look up correct HCL syntax, explore available resources for any provider, and validate configurations against the official registry — all without leaving the conversation context.

This server bridges the gap between natural language infrastructure requests and correct Terraform code generation. Instead of relying on potentially outdated training data, agents can query live documentation for any Terraform provider or module, ensuring generated HCL is accurate and up-to-date.

For teams adopting Infrastructure as Code, this MCP server dramatically reduces the iteration cycle between writing Terraform configs and validating them. It supports both local stdio transport for single-user setups and Streamable HTTP for shared team environments.

## Key Tools
| Tool | Description |
|------|-------------|
| resolveProviderDocID | Resolve a provider documentation identifier from the registry |
| getProviderDoc | Retrieve full provider documentation content |
| resolveModuleDocID | Resolve a module documentation identifier |
| getModuleDoc | Retrieve full module documentation and input/output schemas |
| resolveResourceDocID | Resolve a specific resource type documentation ID |
| getResourceDoc | Retrieve resource documentation including arguments and attributes |
| listProviderResources | List all available resources for a given provider |
| listDataSources | List all available data sources for a given provider |

## Configuration
```json
{
  "mcpServers": {
    "terraform": {
      "command": "npx",
      "args": ["-y", "@hashicorp/terraform-mcp-server"],
      "env": {}
    }
  }
}
```

## Use Cases
1. Generate HCL configurations from natural language descriptions (e.g., "create an AWS VPC with 3 subnets")
2. Look up provider documentation to verify correct argument names and types for resources
3. Explore module schemas to understand required inputs and expected outputs
4. Validate resource configurations against official provider specifications before applying

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Deploy ML training on Kubernetes | tao-run-on-kubernetes | Provision K8s infrastructure with Terraform for TAO training workloads |
| Provision GPU clusters | nemo-automodel-distributed-training | Create GPU compute clusters via Terraform for distributed NeMo training |
| Provision edge infrastructure | jetson-init-target | Define and deploy edge compute infrastructure for Jetson deployments |

## Prerequisites
- Node.js 18+
- Terraform knowledge (HCL syntax, providers, modules)
- Access to Terraform Registry (public or private)

## Security Notes
- Local stdio mode is recommended for individual development use
- Configure `MCP_ALLOWED_ORIGINS` environment variable when using Streamable HTTP transport
- No cloud credentials are transmitted through the MCP server itself — it only serves documentation
- Ensure the server is not exposed to untrusted networks in HTTP mode

## References
- https://github.com/hashicorp/terraform-mcp-server
- https://registry.terraform.io/
- https://developer.hashicorp.com/terraform/docs
