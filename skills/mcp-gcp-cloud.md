# Google Cloud Unified MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Google Cloud Unified MCP Server |
| **Category** | Cloud/Infrastructure |
| **Official** | ⭐ Official (Google Cloud) |
| **Source** | https://github.com/google-cloud/mcp-gcp |
| **Transport** | stdio |
| **Install** | `npx -y @google-cloud/mcp-gcp` |

## Tools & Capabilities
- `list_projects` — Query GCP projects, IDs, and active billing associations
- `manage_cloud_run` — Deploy, inspect revisions, and configure traffic routing for Cloud Run
- `list_gke_clusters` — Inspect GKE Kubernetes clusters, node pools, and versions
- `get_iam_policy` — Audit service account roles and IAM bindings
- `inspect_compute_instances` — Query Compute Engine VM instances and network interfaces

## Client Configuration
```json
{
  "mcpServers": {
    "gcp-cloud": {
      "command": "npx",
      "args": [
        "-y",
        "@google-cloud/mcp-gcp"
      ],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/service-account.json",
        "GCP_PROJECT_ID": "your-project-id"
      }
    }
  }
}
```

## Security & Best Practices
- Use dedicated service account with minimal IAM roles. Never commit service account JSON files.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
