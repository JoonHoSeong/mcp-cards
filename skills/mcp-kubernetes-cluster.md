# Kubernetes MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Kubernetes MCP Server |
| **Category** | Infrastructure/Container Orchestration |
| **Official** | Community (multiple implementations) |
| **Source** | https://github.com/strowk/mcp-k8s-go (Go) / https://github.com/containers/kubernetes-mcp-server (Red Hat) |
| **Transport** | stdio |
| **Install** | `go install` or Docker |

## Description
The Kubernetes MCP Server provides AI assistants with direct access to Kubernetes cluster operations — listing pods, inspecting deployments, reading logs, scaling workloads, and applying manifests. Multiple community implementations exist, with the Go-based `mcp-k8s-go` and Red Hat's `kubernetes-mcp-server` being the most mature options.

This server transforms Kubernetes management from complex kubectl commands into natural language interactions. Instead of remembering exact resource paths, label selectors, and output formats, operators can ask questions like "show me failing pods in the production namespace" or "scale the api-gateway deployment to 5 replicas" and get immediate results.

For GPU-intensive workloads common in NVIDIA AI pipelines, the Kubernetes MCP server is particularly valuable. It enables agents to monitor training jobs, inspect GPU node allocations, troubleshoot OOM kills on model serving pods, and manage the lifecycle of ML workloads — all through conversational interfaces that reduce the cognitive overhead of Kubernetes operations.

## Key Tools
| Tool | Description |
|------|-------------|
| list_pods | List pods in a namespace with status and resource information |
| get_pod | Get detailed information about a specific pod |
| list_deployments | List deployments with replica counts and conditions |
| scale_deployment | Scale a deployment to a specified number of replicas |
| get_logs | Retrieve container logs from a pod |
| list_namespaces | List all namespaces in the cluster |
| apply_manifest | Apply a Kubernetes manifest (YAML/JSON) to the cluster |
| delete_resource | Delete a specific Kubernetes resource |
| get_events | List cluster or namespace events for troubleshooting |
| describe_resource | Get detailed description of any Kubernetes resource |
| list_services | List services with their types and endpoints |
| port_forward | Forward a local port to a pod for debugging |

## Configuration
```json
{
  "mcpServers": {
    "kubernetes": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-v", "${HOME}/.kube/config:/root/.kube/config:ro",
        "ghcr.io/strowk/mcp-k8s-go:latest"
      ]
    }
  }
}
```

## Use Cases
1. Natural language Kubernetes management — query and operate clusters without memorizing kubectl syntax
2. Troubleshoot failing pods — inspect logs, events, and resource limits to diagnose issues
3. Scale deployments — adjust replica counts based on load or scheduling requirements
4. Inspect logs — tail container logs across pods for debugging distributed applications
5. Apply manifests — deploy new workloads or update configurations through conversation

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| TAO training on Kubernetes | tao-run-on-kubernetes | Monitor and manage TAO training pods, check GPU allocation, inspect training logs |
| Distributed training management | nemo-mbridge-multi-node-slurm | Manage multi-node training jobs running on Kubernetes GPU clusters |
| Deploy video analytics services | vss-deploy-profile | Deploy and monitor Video Storage Service profiles on Kubernetes |

## Prerequisites
- Valid kubeconfig file with cluster access credentials
- Kubernetes cluster (local or remote) with API server accessibility
- Docker runtime (for containerized MCP server) or Go toolchain (for source install)

## Security Notes
- Use Kubernetes RBAC to limit the service account's permissions to required namespaces only
- Limit namespace access — bind the MCP server's credentials to specific namespaces, not cluster-admin
- Default to read-only operations; require explicit confirmation for write operations (scale, apply, delete)
- Mount kubeconfig as read-only (`:ro`) when running in Docker
- Audit all mutating operations through Kubernetes audit logs
- Consider using impersonation headers to track which agent initiated each operation

## References
- https://github.com/strowk/mcp-k8s-go
- https://github.com/containers/kubernetes-mcp-server
- https://kubernetes.io/docs/reference/kubectl/
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/
