# Anthropic Local Filesystem MCP Server

## Overview
| Field | Value |
|---|---|
| **Name** | Anthropic Local Filesystem MCP Server |
| **Category** | Core/Filesystem |
| **Official** | ⭐ Official (Anthropic Reference) |
| **Source** | https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem |
| **Transport** | stdio |
| **Install** | `npx -y @modelcontextprotocol/server-filesystem` |

## Tools & Capabilities
- `read_file` — Read complete text or binary contents of file within permitted directory roots
- `write_file` — Create or overwrite file contents with strict path boundary enforcement
- `list_directory` — Traverse directory children with file type and size metadata
- `move_file` — Move or rename files within allowed workspace boundaries
- `search_files` — Find files matching glob patterns and content search queries

## Client Configuration
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/directory"
      ]
    }
  }
}
```

## Security & Best Practices
- Strict directory root path whitelisting. Rejects any path traversal (`../`) outside allowed boundaries.

## NVIDIA Skill & Agent Combinations
| Use Case | Combined NVIDIA Skill / Workflow | Benefit |
|---|---|---|
| AI Agent Workflow | `nvidia-skill-finder` / `rag-blueprint` | Seamless tool calling and automated multi-modal workflow execution |
| Infrastructure Automation | `tao-launch-workflow` / `nim-agent-blueprint` | Automated deployment, resource monitoring, and continuous integration |
