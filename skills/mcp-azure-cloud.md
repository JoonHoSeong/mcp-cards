# Azure MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Azure MCP |
| **Category** | Cloud |
| **Official** | Yes (Microsoft) |
| **Source** | https://github.com/mcp/com.microsoft/azure |
| **Transport** | stdio |
| **Install** | `npx @microsoft/azure-mcp-server` |

## Description
The Azure MCP Server provides AI agents with direct access to Microsoft Azure cloud resources. It enables programmatic management of resource groups, virtual machines, storage accounts, ARM template deployments, and cost analysis — allowing agents to provision, monitor, and optimize Azure infrastructure without leaving the development workflow.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_resource_groups` | List all resource groups in a subscription |
| `create_resource` | Create a new Azure resource with specified configuration |
| `list_vms` | List virtual machines across resource groups |
| `get_vm_status` | Get the running status of a specific VM |
| `deploy_arm_template` | Deploy an Azure Resource Manager template |
| `list_storage_accounts` | List all storage accounts in a subscription |
| `query_cost_management` | Query Azure Cost Management for spending data |

## Configuration
```json
{
  "mcpServers": {
    "azure": {
      "command": "npx",
      "args": ["@microsoft/azure-mcp-server"],
      "env": {
        "AZURE_SUBSCRIPTION_ID": "your-subscription-id",
        "AZURE_TENANT_ID": "your-tenant-id",
        "AZURE_CLIENT_ID": "your-client-id",
        "AZURE_CLIENT_SECRET": "your-client-secret"
      }
    }
  }
}
```

## Use Cases
1. Provision and manage GPU-enabled VMs for distributed model training
2. Monitor infrastructure costs and optimize resource allocation across teams
3. Deploy ARM templates for reproducible ML infrastructure environments

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| GPU cluster provisioning | nemo-automodel-distributed-training | Provision Azure NC/ND-series GPU VMs for multi-node NeMo training |
| Cost optimization | nemo-automodel-distributed-training | Track GPU VM costs during large-scale training runs |
| Infrastructure scaling | nemo-automodel-distributed-training | Scale VM count based on training job requirements |

## Prerequisites
- Azure subscription with active billing
- Azure service principal credentials (client ID, secret, tenant ID)
- Appropriate RBAC roles assigned to the service principal

## Security Notes
- Store Azure credentials in environment variables, never in code
- Use least-privilege RBAC roles for service principals
- Enable Azure AD conditional access for additional security
- Rotate client secrets regularly

## References
- https://github.com/mcp/com.microsoft/azure
- https://learn.microsoft.com/en-us/azure/azure-resource-manager/
- https://learn.microsoft.com/en-us/azure/cost-management-billing/
