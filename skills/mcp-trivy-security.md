# Trivy Container & Infrastructure Security MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Trivy Container & Infrastructure Security MCP Server |
| **Category** | Security/Vulnerability |
| **Official** | ⭐ Official (Aqua Security) |
| **Source** | https://github.com/aquasecurity/mcp-trivy |
| **Transport** | stdio |
| **Install** | `npx -y @aquasecurity/mcp-trivy` |

## Tools & Capabilities
- `scan_container_image` — Scan Docker/OCI container images for OS package and language library CVEs
- `scan_filesystem` — Audit repository filesystem for hardcoded secrets, misconfigurations, and licenses
- `scan_kubernetes_cluster` — Run security posture and compliance assessments on active K8s cluster
- `get_vulnerability_fix` — Retrieve remediated package versions and CVE severity ratings

## Client Configuration
```json
{
  "mcpServers": {
    "trivy": {
      "command": "npx",
      "args": [
        "-y",
        "@aquasecurity/mcp-trivy"
      ]
    }
  }
}
```

## Security & Best Practices
- Local Trivy vulnerability database cache. Offline scanning capability supported.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
