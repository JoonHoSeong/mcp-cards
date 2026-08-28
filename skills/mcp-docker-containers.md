# Docker MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Docker MCP |
| **Category** | Containers |
| **Official** | No (Community) |
| **Source** | https://github.com/QuantGeekDev/docker-mcp |
| **Transport** | stdio |
| **Install** | `npx docker-mcp-server` |

## Description
The Docker MCP Server provides AI agents with full Docker container and image management capabilities. It enables creating, starting, stopping, and inspecting containers, building images, managing Docker Compose stacks, and executing commands inside running containers — essential for orchestrating containerized ML inference pipelines and development environments.

## Key Tools
| Tool | Description |
|------|-------------|
| `list_containers` | List all running and stopped containers |
| `create_container` | Create a new container from an image |
| `start_container` | Start a stopped container |
| `stop_container` | Stop a running container |
| `get_logs` | Retrieve container stdout/stderr logs |
| `list_images` | List locally available Docker images |
| `build_image` | Build an image from a Dockerfile |
| `compose_up` | Start services defined in docker-compose |
| `compose_down` | Stop and remove Compose services |
| `exec_command` | Execute a command in a running container |

## Configuration
```json
{
  "mcpServers": {
    "docker": {
      "command": "npx",
      "args": ["docker-mcp-server"],
      "env": {
        "DOCKER_HOST": "unix:///var/run/docker.sock"
      }
    }
  }
}
```

## Use Cases
1. Build and run containerized DeepStream video analytics pipelines
2. Manage multi-container ML serving stacks with Docker Compose
3. Debug inference containers by executing diagnostic commands

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Pipeline containers | deepstream-generate-pipeline | Run DeepStream pipelines in GPU-enabled containers |
| Build images | deepstream-generate-pipeline | Build custom DeepStream container images |
| Stack management | deepstream-generate-pipeline | Orchestrate multi-container inference stacks |

## Prerequisites
- Docker daemon running and accessible
- Docker socket permissions for the executing user
- NVIDIA Container Toolkit for GPU workloads (optional)

## Security Notes
- Docker socket access grants root-equivalent privileges
- Use rootless Docker mode where possible
- Restrict container capabilities with security profiles
- Never expose Docker socket over TCP without TLS

## References
- https://github.com/QuantGeekDev/docker-mcp
- https://docs.docker.com/engine/api/
- https://docs.docker.com/compose/
