# ArgoCD MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | ArgoCD MCP |
| **Category** | GitOps/CD |
| **Official** | Yes (Argo Project Labs) |
| **Source** | https://github.com/akuity/argocd-mcp |
| **Transport** | stdio |
| **Install** | `npx @akuity/argocd-mcp-server` |

## Description
The ArgoCD MCP Server provides AI agents with GitOps continuous delivery capabilities through Argo CD. It enables managing applications, monitoring sync status, triggering synchronizations, and performing rollbacks — allowing agents to implement declarative, Git-driven deployments for Kubernetes-based ML model serving and training infrastructure.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_applications` | List all ArgoCD applications |
| `get_app_status` | Get the sync and health status of an app |
| `sync_application` | Trigger a sync to match desired state |
| `get_app_health` | Get detailed health information |
| `list_repositories` | List configured Git repositories |
| `get_sync_history` | Retrieve sync operation history |
| `rollback_application` | Rollback to a previous revision |
| `get_resource_tree` | Get the resource dependency tree |

## Configuration
```json
{
  "mcpServers": {
    "argocd": {
      "command": "npx",
      "args": ["@akuity/argocd-mcp-server"],
      "env": {
        "ARGOCD_SERVER": "https://argocd.your-domain.com",
        "ARGOCD_AUTH_TOKEN": "your-auth-token"
      }
    }
  }
}
```

## Use Cases
1. Deploy ML model serving infrastructure via Git-driven workflows
2. Monitor and auto-sync Kubernetes deployments for training pipelines
3. Rollback failed model deployments to last known good state

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| GitOps ML deploy | tao-run-on-kubernetes | Deploy TAO training jobs via GitOps to Kubernetes |
| Rollback models | tao-run-on-kubernetes | Roll back failed model training deployments |
| Sync monitoring | tao-run-on-kubernetes | Monitor sync status of ML pipeline manifests |

## Prerequisites
- ArgoCD instance deployed on Kubernetes
- `ARGOCD_SERVER` URL of the ArgoCD API server
- `ARGOCD_AUTH_TOKEN` for API authentication

## Security Notes
- Use project-scoped tokens to limit application access
- Enable SSO integration for user authentication
- Restrict sync permissions by namespace and cluster
- Audit all sync operations through ArgoCD's event log

## References
- https://github.com/akuity/argocd-mcp
- https://argo-cd.readthedocs.io/en/stable/
- https://argo-cd.readthedocs.io/en/stable/user-guide/
