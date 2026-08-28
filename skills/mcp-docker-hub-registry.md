# Docker Hub MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Docker Hub MCP |
| **Category** | Container Registry |
| **Official** | Yes (Docker) |
| **Source** | https://github.com/docker/hub-mcp |
| **Transport** | stdio |
| **Install** | `npx @docker/hub-mcp-server` |

## Description
The Docker Hub MCP Server provides AI agents with access to Docker Hub's container registry API. It enables searching for images, inspecting repository details, listing tags, checking vulnerability reports, and browsing organizations — allowing agents to discover and validate container images for NVIDIA DPU workloads, ML frameworks, and inference runtimes.

## Key Tools
| Tool | Description |
|------|-------------|
| `search_images` | Search Docker Hub for container images |
| `get_repository` | Get detailed repository information |
| `list_tags` | List available tags for a repository |
| `get_tag_details` | Get manifest and layer details for a tag |
| `list_organizations` | List organizations the user belongs to |
| `get_vulnerability_report` | Get CVE vulnerability scan results |
| `list_namespaces` | List accessible namespaces |

## Configuration
```json
{
  "mcpServers": {
    "docker-hub": {
      "command": "npx",
      "args": ["@docker/hub-mcp-server"],
      "env": {
        "DOCKER_HUB_TOKEN": "your-docker-hub-token"
      }
    }
  }
}
```

## Use Cases
1. Discover and validate NVIDIA DPU container images for DOCA deployments
2. Check vulnerability reports before pulling base images for ML pipelines
3. List and compare tagged versions of inference runtime containers

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| DPU image discovery | doca-container-deployment | Find official NVIDIA DOCA container images |
| Image validation | doca-container-deployment | Verify image tags and vulnerability status before deploy |
| Version management | doca-container-deployment | Track available DPU container versions |

## Prerequisites
- Docker Hub account (free or Pro)
- `DOCKER_HUB_TOKEN` personal access token
- Docker Scout enabled for vulnerability scanning (optional)

## Security Notes
- Use read-only tokens for image discovery operations
- Verify image signatures before deployment to production
- Check vulnerability reports for critical CVEs before pulling
- Prefer official and verified publisher images

## References
- https://github.com/docker/hub-mcp
- https://docs.docker.com/docker-hub/api/
- https://docs.docker.com/scout/
