# Pulumi MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Pulumi MCP |
| **Category** | Infrastructure/IaC |
| **Official** | Yes (Pulumi) |
| **Source** | https://github.com/pulumi/mcp-server |
| **Transport** | stdio |
| **Install** | `npx @pulumi/mcp-server` |

## Description
The Pulumi MCP Server enables AI agents to manage infrastructure as code using Pulumi's multi-language SDK. It provides tools for previewing and deploying infrastructure changes, managing stacks, and querying outputs — allowing agents to provision cloud resources using Python, TypeScript, Go, or C# with full state management and drift detection.

## Key Tools
| Tool | Description |
|------|-------------|
| `get_package_info` | Get information about a Pulumi package/provider |
| `preview_update` | Preview infrastructure changes without deploying |
| `deploy_update` | Deploy infrastructure changes to the target stack |
| `get_stack_outputs` | Retrieve outputs from a deployed stack |
| `list_stacks` | List all stacks in a project |
| `get_stack_state` | Get the current state of a stack's resources |
| `destroy_stack` | Destroy all resources in a stack |
| `refresh_stack` | Refresh stack state from cloud provider |

## Configuration
```json
{
  "mcpServers": {
    "pulumi": {
      "command": "npx",
      "args": ["@pulumi/mcp-server"],
      "env": {
        "PULUMI_ACCESS_TOKEN": "your-pulumi-token",
        "PULUMI_ORG": "your-organization"
      }
    }
  }
}
```

## Use Cases
1. Provision multi-GPU Kubernetes clusters for distributed training workloads
2. Preview and deploy infrastructure changes with automated drift detection
3. Manage multi-cloud AI infrastructure stacks across AWS, Azure, and GCP

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| GPU cluster provisioning | nemo-automodel-distributed-training | Define GPU cluster infrastructure in Python/TypeScript code |
| Multi-node setup | nemo-automodel-distributed-training | Provision networking and storage for distributed training |
| Environment management | nemo-automodel-distributed-training | Manage dev/staging/prod training environments as stacks |

## Prerequisites
- Pulumi account with organization access
- `PULUMI_ACCESS_TOKEN` for authentication
- Cloud provider credentials (AWS/Azure/GCP) configured locally

## Security Notes
- Use Pulumi ESC for centralized secrets management
- Enable stack policies to prevent destructive operations
- Preview all changes before deploying to production stacks
- Use RBAC to restrict who can deploy to production

## References
- https://github.com/pulumi/mcp-server
- https://www.pulumi.com/docs/
- https://www.pulumi.com/docs/iac/concepts/stacks/
